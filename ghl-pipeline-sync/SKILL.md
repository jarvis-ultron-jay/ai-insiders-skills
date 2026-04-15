# GHL Pipeline Sync

**Version:** 1.0.0
**Description:** Connect to GoHighLevel (GHL) API, authenticate with PIT tokens, pull pipeline and opportunity data with correct pagination, and sync everything to Supabase PostgreSQL. Covers all 9 Wild Success sub-accounts, rate limiting, error handling, stage mapping, PIF detection via contact tags, and cursor-based resume for crash recovery.

**Trigger phrases:** "GHL sync", "pipeline sync", "sync GHL to Supabase", "pull opportunities", "GHL API", "GHL pagination", "pipeline data", "opportunity sync", "GHL contacts", "sub-account sync", "PIT token"

---

## Context: GoHighLevel at Wild Success

GoHighLevel (GHL) is Wild Success's CRM platform. It's organised into **sub-accounts** (called "locations"), each with its own data, pipelines, contacts, and API token. Wild Success has 9 sub-accounts — one for each programme vertical.

**Why this skill exists:** GHL's native reporting is limited. To build dashboards, run audits, and generate reports, we sync GHL data to Supabase (PostgreSQL) where it can be queried, joined, and visualised.

---

## Section 1: Authentication

### 1.1 PIT Tokens (Private Integration Tokens)

Each GHL sub-account requires its own PIT (Private Integration Token). PITs are NOT shared across sub-accounts.

**Where tokens are stored:** Supabase credential store (`agent_credentials` table in the `wild-success-tools` project). Query by `service_name = 'ghl'` and `credential_key = 'pit_<location_slug>'`.

**If using environment variables or config files:** Tokens may also be in `openclaw.json` or environment variables. Never write PIT values into workspace files, logs, or SKILL.md files.

### 1.2 API Versions

GHL has two API versions:

| Version | Base URL | Auth Header | Version Header | Notes |
|---------|----------|-------------|----------------|-------|
| v1 | `https://rest.gohighlevel.com/v1` | `Authorization: Bearer <location-key>` | None | Legacy, fewer endpoints |
| **v2** | `https://services.leadconnectorhq.com` | `Authorization: Bearer <PIT>` | `Version: 2021-07-28` | Current, use this one |

**Always use v2.** All endpoints in this skill use v2 unless noted.

### 1.3 Required Headers (Every Request)

```
Authorization: Bearer <PIT_TOKEN>
Version: 2021-07-28
Content-Type: application/json
```

Missing the `Version` header will return 400 or unexpected results.

---

## Section 2: Sub-Account Structure

### 2.1 The 9 Sub-Accounts

| Sub-Account Name | Location ID | Status | Notes |
|-----------------|-------------|--------|-------|
| **Wild Success** (WILD Central) | `POOwEAaRUV4mWkn72yBa` | Active | Main account. Business Coaching, MOC pipelines live here. Cross-account sales (see §6.1). |
| **Human Design** | `7uhGOEE0XXWrxiXHPv0o` | Active | HD programme pipelines |
| **Breathwork** | `OAhLgH9zYEVCYYRuRpCN` | Active | Breathwork programme |
| **AI** | `zFOxEl40KGPj5B7ETDJM` | Active | AI programme pipelines |
| **SMMC** | `Xebha2i71m3aEkhNYPAh` | Active | Social Media Marketing |
| **Personal Branding** | `z33PQnGmPgH7xyrNqnBd` | Active | Personal Branding programme |
| **Meditation** | `GmLZzsWRaH0s31XCwKPR` | Active | Meditation programme |
| **Astrology** | *(confirm with Jaydyn)* | Active | Astrology programme |
| **Biohacking** | *(confirm with Jaydyn)* | Low activity | May be empty |

### 2.2 Why Multiple Sub-Accounts Matter

- Each sub-account has **separate contacts, pipelines, tags, and workflows**
- A person can be a contact in multiple sub-accounts with different tags
- **Business Coaching** is always sold via WILD Central, even when pitched during another programme's launch. If you only sync the programme's sub-account, you WILL miss Business Coaching sales.
- A complete sync must iterate over ALL active sub-accounts

---

## Section 3: API Endpoints

### 3.1 List Pipelines

```
GET https://services.leadconnectorhq.com/opportunities/pipelines
```

Query params: `locationId=<LOCATION_ID>`

Response:
```json
{
  "pipelines": [
    {
      "id": "pipeline_abc123",
      "name": "Business Coaching (March 2026)",
      "stages": [
        {"id": "stage_1", "name": "New Lead", "position": 0},
        {"id": "stage_2", "name": "Booked", "position": 1},
        {"id": "stage_3", "name": "Won", "position": 5}
      ]
    }
  ]
}
```

### 3.2 Search Opportunities (Primary Sync Endpoint)

```
POST https://services.leadconnectorhq.com/opportunities/search
```

Request body:
```json
{
  "location_id": "<LOCATION_ID>",
  "pipeline_id": "<PIPELINE_ID>",
  "limit": 100,
  "startAfterId": "<last_opp_id>",
  "startAfter": "<last_opp_timestamp>"
}
```

Response:
```json
{
  "opportunities": [
    {
      "id": "opp_abc123",
      "name": "John Smith",
      "monetaryValue": 7800,
      "pipelineId": "pipeline_abc123",
      "pipelineStageId": "stage_won",
      "assignedTo": "Daniel Lewis",
      "status": "open",
      "contact": {
        "id": "contact_xyz",
        "name": "John Smith",
        "email": "john@example.com",
        "tags": ["biz coaching | pif | $7800", "nlpftmar26"]
      },
      "createdAt": "2026-03-25T10:30:00.000Z",
      "updatedAt": "2026-03-26T14:20:00.000Z"
    }
  ],
  "meta": {
    "total": 289,
    "nextPageUrl": "..."
  }
}
```

### 3.3 Get Single Contact

```
GET https://services.leadconnectorhq.com/contacts/<CONTACT_ID>
```

Query params: `locationId=<LOCATION_ID>` (if not embedded in token scope)

Response includes `contact.tags` array — needed for PIF detection.

### 3.4 Get Single Opportunity

```
GET https://services.leadconnectorhq.com/opportunities/<OPPORTUNITY_ID>
```

---

## Section 4: Pagination (CRITICAL)

> ⚠️ **WARNING — THIS IS THE #1 GOTCHA IN THE GHL API**
>
> GHL v2 opportunity search requires **BOTH** `startAfterId` AND `startAfter` for pagination. Using only one parameter returns page 1 forever. You will think you're paginating (the response looks normal, you get 100 results) but you're stuck on page 1 in an infinite loop.

### 4.1 Correct Pagination

```python
def fetch_all_opportunities(location_id, pipeline_id, pit_token):
    """
    Fetches ALL opportunities from a GHL pipeline with correct pagination.
    
    CRITICAL: Must use BOTH startAfterId AND startAfter.
    meta.nextPageUrl is UNRELIABLE — do not use it.
    """
    all_opps = []
    start_after_id = None
    start_after = None
    page = 0
    seen_first_ids = set()  # Track first IDs to detect stuck pagination
    
    while True:
        page += 1
        payload = {
            "location_id": location_id,
            "pipeline_id": pipeline_id,
            "limit": 100
        }
        
        if start_after_id and start_after:
            payload["startAfterId"] = start_after_id
            payload["startAfter"] = start_after
        
        response = rate_limited_request(
            url="https://services.leadconnectorhq.com/opportunities/search",
            headers={
                "Authorization": f"Bearer {pit_token}",
                "Version": "2021-07-28",
                "Content-Type": "application/json"
            },
            json_payload=payload
        )
        
        opps = response.get("opportunities", [])
        
        if not opps:
            break
        
        # SAFETY CHECK: Detect stuck pagination
        first_id = opps[0]["id"]
        if first_id in seen_first_ids:
            raise Exception(
                f"PAGINATION STUCK: Page {page} returned same first ID {first_id}. "
                f"Check startAfterId and startAfter params."
            )
        seen_first_ids.add(first_id)
        
        all_opps.extend(opps)
        
        # Set cursors for next page
        last_opp = opps[-1]
        start_after_id = last_opp["id"]
        start_after = last_opp.get("updatedAt") or last_opp.get("lastUpdatedAt")
        
        if not start_after:
            raise Exception(
                f"Cannot determine startAfter timestamp from last opportunity. "
                f"Available fields: {list(last_opp.keys())}"
            )
        
        # If fewer than limit returned, this is the last page
        if len(opps) < 100:
            break
    
    return all_opps
```

### 4.2 Pagination Debugging

If you suspect pagination is broken:

1. Log `page_number`, `first_opp_id`, `last_opp_id`, `count` for each page
2. If `first_opp_id` repeats → pagination parameters are wrong
3. Verify both `startAfterId` and `startAfter` are being sent
4. Verify `startAfter` is the timestamp, not the ID (they look different — IDs are alphanumeric, timestamps are ISO 8601)
5. Try using `createdAt` instead of `updatedAt` for the `startAfter` value if `updatedAt` is missing

### 4.3 meta.nextPageUrl

The response includes `meta.nextPageUrl`. **Do not rely on it.** It is inconsistently populated across GHL API versions and sometimes returns `null` even when more pages exist. Always use the cursor-based approach above.

---

## Section 5: Rate Limiting & Error Handling

### 5.1 Rate Limits

| Limit | Value |
|-------|-------|
| Requests per minute per location | ~100 |
| HTTP status on limit | 429 Too Many Requests |
| Recommended delay between requests | 0.7 seconds (~85 req/min) |

### 5.2 Rate Limiter Implementation

```python
import time
import random
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def create_session():
    """Create a requests session with retry logic."""
    session = requests.Session()
    retry = Retry(
        total=5,
        backoff_factor=1,
        status_forcelist=[429, 500, 502, 503, 504]
    )
    adapter = HTTPAdapter(max_retries=retry)
    session.mount("https://", adapter)
    return session

def rate_limited_request(url, headers, json_payload, max_retries=5):
    """Make a rate-limited request with exponential backoff."""
    session = create_session()
    
    for attempt in range(max_retries):
        try:
            response = session.post(url, headers=headers, json=json_payload)
            
            if response.status_code == 429:
                wait = min(60, (2 ** attempt) + random.uniform(0, 1))
                print(f"Rate limited. Waiting {wait:.1f}s (attempt {attempt + 1})")
                time.sleep(wait)
                continue
            
            response.raise_for_status()
            time.sleep(0.7)  # Proactive rate limiting
            return response.json()
            
        except requests.exceptions.SSLError:
            # UNEXPECTED_EOF_WHILE_READING — create new session
            print(f"SSL error, creating new session (attempt {attempt + 1})")
            session = create_session()
            time.sleep(2 ** attempt)
            continue
            
        except requests.exceptions.ConnectionError:
            print(f"Connection error (attempt {attempt + 1})")
            time.sleep(2 ** attempt)
            continue
    
    raise Exception(f"Max retries ({max_retries}) exceeded for {url}")
```

### 5.3 SSL Error Recovery

GHL connections sometimes fail with `UNEXPECTED_EOF_WHILE_READING`. This happens when:
- Rate limiting causes the server to drop the connection
- Network instability during sustained API sessions
- Server-side timeouts on long-running requests

**Fix:** Create a NEW HTTP session (don't reuse the broken one), sleep with backoff, retry.

---

## Section 6: Stage Mapping

### 6.1 What Stage Maps Are

A stage map translates GHL stage IDs to categories (won/open/lost). Without a stage map, you can't determine which opportunities are actual sales.

### 6.2 Stage Map Format

```json
{
  "pipeline_id": "abc123",
  "pipeline_name": "Business Coaching (March 2026)",
  "location_id": "POOwEAaRUV4mWkn72yBa",
  "programme": "Business Coaching",
  "stages": {
    "stage_abc1": {"name": "New Lead", "category": "open"},
    "stage_abc2": {"name": "Booked", "category": "open"},
    "stage_abc3": {"name": "Showed", "category": "open"},
    "stage_abc4": {"name": "Not Yet Called", "category": "open"},
    "stage_abc5": {"name": "Won", "category": "won"},
    "stage_abc6": {"name": "PIF", "category": "won"},
    "stage_abc7": {"name": "Payment Plan", "category": "won"},
    "stage_abc8": {"name": "Future PP", "category": "won"},
    "stage_abc9": {"name": "Lost", "category": "lost"},
    "stage_abc10": {"name": "No Show", "category": "lost"},
    "stage_abc11": {"name": "RTC", "category": "lost"}
  }
}
```

### 6.3 Building a Stage Map

```bash
# 1. List all pipelines in a location
curl -s "https://services.leadconnectorhq.com/opportunities/pipelines?locationId=<LOCATION_ID>" \
  -H "Authorization: Bearer <PIT>" \
  -H "Version: 2021-07-28" | jq '.pipelines[] | {id, name, stages: [.stages[] | {id, name}]}'

# 2. For each pipeline, assign categories to each stage
# Won stages: typically named "Won", "PIF", "Payment Plan", "Future PP", "Sold"
# Lost stages: typically named "Lost", "RTC", "No Show", "DQ", "Not Interested"
# Open stages: everything else (New Lead, Booked, Showed, Not Yet Called, etc.)
```

> ⚠️ **WARNING — REGRESSION T-11: Silent Stage Drops**
>
> If a GHL stage exists but is NOT in your local stage map, opportunities in that stage are **completely invisible**. They are fetched from the API but silently skipped during processing because the stage has no category.
>
> **This is not an error — it's silent data loss.**
>
> **Real incident (L3):** A "Future PP" stage was added to all 6 GHL pipelines mid-campaign. It was missing from all local stage maps. Result:
> - 6 Wellness sales (all from rep Caroline Hayner) were invisible
> - Wins in other programmes were also missed
> - Revenue was understated
> - Rep leaderboard was wrong (Caroline appeared to have 0 sales when she actually had 13+)
> - Discovered only when manual GHL count didn't match dashboard
>
> **Prevention:**
> 1. After ANY pipeline change in GHL, re-pull all stages and update the local map
> 2. During each sync, log any opportunity whose `pipelineStageId` is NOT in the stage map
> 3. Run a daily verification: count opps per stage in GHL vs. your processed data

### 6.4 Stage Map Verification Script

```python
def verify_stage_map(opportunities, stage_map):
    """Check for opportunities in stages not in the local map."""
    unknown_stages = {}
    for opp in opportunities:
        stage_id = opp.get("pipelineStageId")
        if stage_id not in stage_map["stages"]:
            unknown_stages[stage_id] = unknown_stages.get(stage_id, 0) + 1
    
    if unknown_stages:
        print("⚠️ UNKNOWN STAGES DETECTED:")
        for stage_id, count in unknown_stages.items():
            print(f"  Stage {stage_id}: {count} opportunities DROPPED")
        print("ACTION REQUIRED: Update stage map immediately.")
        return False
    return True
```

---

## Section 7: PIF Detection

> ⚠️ **WARNING — NEVER USE DOLLAR THRESHOLD FOR PIF DETECTION**
>
> The original method was: if `monetaryValue >= $3,000` → PIF. This is WRONG.
>
> **Why it fails:**
> - Scholarship PIFs have values of $2,000 (below any reasonable threshold)
> - Some payment plans have high initial values that look like PIFs
> - During L3, dollar-threshold detection was $400 off vs. tag-based detection
>
> **Correct method:** Look up GHL contact tags.

### 7.1 PIF Tag Lookup

```python
def detect_pif(contact_id, programme_name, pit_token, location_id):
    """
    Detect PIF/PP status from contact tags.
    
    Returns: ('pif', True) or ('pp', False)
    """
    response = rate_limited_request(
        url=f"https://services.leadconnectorhq.com/contacts/{contact_id}",
        headers={
            "Authorization": f"Bearer {pit_token}",
            "Version": "2021-07-28"
        },
        json_payload=None  # GET request
    )
    
    tags = response.get("contact", {}).get("tags", [])
    
    # Look for programme-specific PIF/PP tags
    # Tag formats vary: "biz coaching | pif | $7800", "advprac | pif | $2000"
    for tag in tags:
        tag_lower = tag.lower()
        if "pif" in tag_lower:
            return ("pif", True)
        if "pp" in tag_lower and ("payment" in tag_lower or "|" in tag_lower):
            return ("pp", False)
    
    # No matching tag found — default to PP
    return ("pp_default", False)
```

### 7.2 PIF Tag Patterns by Programme

| Programme | PIF Tag Pattern | PP Tag Pattern |
|-----------|----------------|----------------|
| Business Coaching | `biz coaching \| pif \| $7800` | `biz coaching \| pp \| $250` |
| Masters of Coaching | `moc \| pif \| $7000` | `moc \| pp \| $220` |
| Human Design | `hd \| pif \| $6700` | `hd \| pp \| $210` |
| AI | `ai \| pif \| $6800` | `ai \| pp \| $215` |
| Astrology | `astro \| pif \| $4700` | `astro \| pp \| $150` |
| Wellness | `wellness \| pif \| $4600` | `wellness \| pp \| $145` |
| Advanced Practitioner | `advprac \| pif \| $2000` | `advprac \| pp \| $50` |

**Note:** Tag patterns may vary slightly between launches. Always verify the exact tag format by checking a few known contacts in GHL before starting a bulk PIF scan.

### 7.3 Optimisation: Batch PIF Detection

Rather than making a separate API call for every won opportunity's contact, batch-fetch contacts:

```python
# If opportunity.contact.tags is included in the search response, use it directly
# Otherwise, collect unique contact IDs and fetch them in batch
contact_ids = set(opp["contact"]["id"] for opp in won_opportunities)
# Fetch each contact (no bulk endpoint exists — GHL requires individual calls)
# Budget: ~100 req/min, so 100 contacts = ~70 seconds
```

---

## Section 8: Supabase Sync

### 8.1 Connection

```python
import psycopg2

def get_connection(supabase_host, supabase_password):
    """
    Connect to Supabase PostgreSQL.
    
    CRITICAL: IPv6 can cause timeouts on some networks. Force IPv4 if needed.
    """
    return psycopg2.connect(
        host=supabase_host,  # e.g., "db.xxxxxxxxxxxx.supabase.co"
        port=5432,
        dbname="postgres",
        user="postgres",
        password=supabase_password,
        sslmode="require",
        connect_timeout=10
    )
```

### 8.2 Upsert Pattern

```python
def upsert_opportunities(conn, opportunities):
    """
    Upsert opportunities to Supabase.
    
    CRITICAL: Always conn.commit() after writes.
    PostgreSQL silently rolls back uncommitted transactions on close.
    """
    cursor = conn.cursor()
    
    upsert_sql = """
    INSERT INTO opportunities (
        opportunity_id, contact_id, contact_name, contact_email,
        pipeline_id, pipeline_name, stage_id, stage_name, stage_category,
        programme, launch, monetary_value, currency, assigned_to,
        is_pif, pif_source, location_id, created_at, updated_at, synced_at
    ) VALUES (
        %(opportunity_id)s, %(contact_id)s, %(contact_name)s, %(contact_email)s,
        %(pipeline_id)s, %(pipeline_name)s, %(stage_id)s, %(stage_name)s, %(stage_category)s,
        %(programme)s, %(launch)s, %(monetary_value)s, %(currency)s, %(assigned_to)s,
        %(is_pif)s, %(pif_source)s, %(location_id)s, %(created_at)s, %(updated_at)s, NOW()
    )
    ON CONFLICT (opportunity_id) DO UPDATE SET
        stage_id = EXCLUDED.stage_id,
        stage_name = EXCLUDED.stage_name,
        stage_category = EXCLUDED.stage_category,
        monetary_value = EXCLUDED.monetary_value,
        assigned_to = EXCLUDED.assigned_to,
        is_pif = EXCLUDED.is_pif,
        pif_source = EXCLUDED.pif_source,
        updated_at = EXCLUDED.updated_at,
        synced_at = NOW()
    """
    
    batch_size = 100
    for i in range(0, len(opportunities), batch_size):
        batch = opportunities[i:i + batch_size]
        for opp in batch:
            cursor.execute(upsert_sql, opp)
        
        conn.commit()  # COMMIT EVERY BATCH — do not skip this
        print(f"Committed batch {i // batch_size + 1} ({len(batch)} records)")
    
    cursor.close()
```

### 8.3 Cursor-Based Resume

For crash recovery, save sync progress:

```sql
CREATE TABLE IF NOT EXISTS sync_cursors (
    source TEXT NOT NULL,
    location_id TEXT NOT NULL,
    pipeline_id TEXT,
    cursor_id TEXT,        -- Last opportunity ID processed
    cursor_ts TEXT,        -- Last opportunity timestamp processed
    records_synced INTEGER DEFAULT 0,
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (source, location_id, pipeline_id)
);
```

```python
def save_cursor(conn, source, location_id, pipeline_id, last_opp_id, last_opp_ts, count):
    cursor = conn.cursor()
    cursor.execute("""
        INSERT INTO sync_cursors (source, location_id, pipeline_id, cursor_id, cursor_ts, records_synced, updated_at)
        VALUES (%s, %s, %s, %s, %s, %s, NOW())
        ON CONFLICT (source, location_id, pipeline_id) DO UPDATE SET
            cursor_id = EXCLUDED.cursor_id,
            cursor_ts = EXCLUDED.cursor_ts,
            records_synced = EXCLUDED.records_synced,
            updated_at = NOW()
    """, (source, location_id, pipeline_id, last_opp_id, last_opp_ts, count))
    conn.commit()

def load_cursor(conn, source, location_id, pipeline_id):
    cursor = conn.cursor()
    cursor.execute("""
        SELECT cursor_id, cursor_ts, records_synced
        FROM sync_cursors
        WHERE source = %s AND location_id = %s AND pipeline_id = %s
    """, (source, location_id, pipeline_id))
    row = cursor.fetchone()
    return row if row else (None, None, 0)
```

### 8.4 Re-Run Rule

> ⚠️ **WARNING — NEVER RE-RUN COMPLETED WORK**
>
> If a sync fails on ONE source (e.g., one sub-account), write a TARGETED script for ONLY that source. Do NOT re-run the entire sync pipeline. Skip to the failure point using cursors or row counts from the database.
>
> Re-running completed sources wastes hours and API calls on data already synced. This has been flagged multiple times.

---

## Section 9: Revenue Calculation

### 9.1 Monetary Value Field

The `monetaryValue` field on opportunities represents the deal value. At Wild Success, this is in AUD.

```python
def extract_revenue(opportunity):
    """
    Extract revenue from an opportunity.
    
    GHL monetaryValue may be stored as dollars (7800) or cents (780000).
    Wild Success programmes range $2,000-$8,000 AUD.
    """
    value = opportunity.get("monetaryValue", 0)
    if value is None:
        return 0
    
    value = float(value)
    
    # Sanity check: if value > 50,000, it's probably in cents
    if value > 50000:
        value = value / 100
    
    return value
```

### 9.2 Currency

All Wild Success revenue is tracked in AUD. For USD reporting, use approximate conversion rate of 0.667 (confirm current rate).

```python
aud_to_usd = 0.667  # Approximate — verify current rate
revenue_usd = revenue_aud * aud_to_usd
```

---

## Section 10: Complete Sync Workflow

### 10.1 Full Pipeline Sync (All Sub-Accounts)

```python
def full_sync(launch_name, pipeline_configs, supabase_conn):
    """
    Sync ALL pipelines across ALL sub-accounts for a launch.
    
    pipeline_configs: list of {
        location_id, pit_token, pipeline_id, pipeline_name, programme, stage_map
    }
    """
    total_synced = 0
    errors = []
    
    for config in pipeline_configs:
        try:
            print(f"\n--- Syncing {config['programme']} ---")
            print(f"Location: {config['location_id']}")
            print(f"Pipeline: {config['pipeline_name']} ({config['pipeline_id']})")
            
            # Check for resume cursor
            cursor_id, cursor_ts, prev_count = load_cursor(
                supabase_conn,
                source="ghl_pipeline",
                location_id=config["location_id"],
                pipeline_id=config["pipeline_id"]
            )
            
            if cursor_id:
                print(f"Resuming from cursor: {cursor_id} ({prev_count} previously synced)")
            
            # Fetch all opportunities
            opportunities = fetch_all_opportunities(
                location_id=config["location_id"],
                pipeline_id=config["pipeline_id"],
                pit_token=config["pit_token"]
            )
            
            print(f"Fetched {len(opportunities)} opportunities")
            
            # Verify stage map completeness
            stage_map_ok = verify_stage_map(opportunities, config["stage_map"])
            if not stage_map_ok:
                errors.append(f"{config['programme']}: Unknown stages detected")
            
            # Process opportunities
            processed = []
            for opp in opportunities:
                stage_id = opp.get("pipelineStageId")
                stage_info = config["stage_map"]["stages"].get(stage_id, {})
                
                if not stage_info:
                    continue  # Unknown stage — already logged by verify_stage_map
                
                # PIF detection for won opportunities
                is_pif = False
                pif_source = "none"
                if stage_info.get("category") == "won":
                    contact_id = opp.get("contact", {}).get("id")
                    if contact_id:
                        pif_source, is_pif = detect_pif(
                            contact_id, config["programme"],
                            config["pit_token"], config["location_id"]
                        )
                
                processed.append({
                    "opportunity_id": opp["id"],
                    "contact_id": opp.get("contact", {}).get("id"),
                    "contact_name": opp.get("contact", {}).get("name", opp.get("name")),
                    "contact_email": opp.get("contact", {}).get("email"),
                    "pipeline_id": config["pipeline_id"],
                    "pipeline_name": config["pipeline_name"],
                    "stage_id": stage_id,
                    "stage_name": stage_info.get("name"),
                    "stage_category": stage_info.get("category"),
                    "programme": config["programme"],
                    "launch": launch_name,
                    "monetary_value": extract_revenue(opp),
                    "currency": "AUD",
                    "assigned_to": normalise_rep_name(opp.get("assignedTo")),
                    "is_pif": is_pif,
                    "pif_source": pif_source,
                    "location_id": config["location_id"],
                    "created_at": opp.get("createdAt"),
                    "updated_at": opp.get("updatedAt")
                })
            
            # Upsert to Supabase
            upsert_opportunities(supabase_conn, processed)
            
            # Save cursor
            if opportunities:
                last = opportunities[-1]
                save_cursor(
                    supabase_conn,
                    source="ghl_pipeline",
                    location_id=config["location_id"],
                    pipeline_id=config["pipeline_id"],
                    last_opp_id=last["id"],
                    last_opp_ts=last.get("updatedAt"),
                    count=len(processed)
                )
            
            won_count = len([p for p in processed if p["stage_category"] == "won"])
            revenue = sum(p["monetary_value"] for p in processed if p["stage_category"] == "won")
            pifs = len([p for p in processed if p["is_pif"]])
            
            print(f"✅ {config['programme']}: {won_count} sales, ${revenue:,.0f} AUD, {pifs} PIFs")
            total_synced += len(processed)
            
        except Exception as e:
            error_msg = f"{config['programme']}: {str(e)}"
            errors.append(error_msg)
            print(f"❌ {error_msg}")
            continue  # Don't stop the entire sync for one failure
    
    print(f"\n=== SYNC COMPLETE ===")
    print(f"Total records synced: {total_synced}")
    if errors:
        print(f"⚠️ Errors ({len(errors)}):")
        for e in errors:
            print(f"  - {e}")
    
    return total_synced, errors
```

### 10.2 Verification After Sync

Always verify after a sync:

```sql
-- Count by programme and stage category
SELECT programme, stage_category, COUNT(*), SUM(monetary_value)
FROM opportunities
WHERE launch = 'L3'
GROUP BY programme, stage_category
ORDER BY programme, stage_category;

-- Compare total won count with GHL manual check
-- (spot-check 2-3 pipelines by counting in GHL UI)
```

---

## Section 11: Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Same data every page (infinite loop) | Missing `startAfterId` or `startAfter` in pagination | Send BOTH parameters (§4) |
| 0 records returned when pipeline has data | Wrong pipeline ID or location ID | Verify IDs with pipeline list endpoint |
| 429 Too Many Requests | Rate limit exceeded | Increase delay between requests (§5) |
| SSL UNEXPECTED_EOF_WHILE_READING | Server dropped connection | Create new session, retry with backoff (§5.3) |
| Data in Supabase disappears after sync | Missing `conn.commit()` | Always commit after writes (§8.2) |
| Some sales missing from count | Stage not in local stage map | Check for unknown stages (§6 / regression T-11) |
| PIF count wrong | Using dollar threshold instead of tags | Use contact tag lookup (§7) |
| Wrong revenue numbers | `monetaryValue` in cents not dollars | Check value range, divide by 100 if needed (§9) |
| Partial sync data | Previous sync crashed mid-run | Use cursor-based resume (§8.3) |

---

*This skill is self-contained. A fresh agent with GHL PIT tokens and Supabase credentials can execute a complete pipeline sync using only this document.*
