---
name: launch-campaign-management
description: End-to-end launch campaign management — pipeline setup, daily syncs, briefs, rep leaderboards, pacing dashboards, and campaign close-out with archival. Covers GHL API, Supabase sync, PIF detection, and all known regressions.
---
# Launch Campaign Management

**Version:** 1.0.0
**Description:** End-to-end management of a Wild Success launch campaign — from pre-launch pipeline setup through daily syncs, daily briefs, rep leaderboards, pacing dashboards, and campaign close-out with archival. Covers every operational detail needed to run a multi-programme, multi-rep sales launch pulling data from GoHighLevel (GHL) and syncing to Supabase.

**Trigger phrases:** "launch campaign", "new launch", "L4 setup", "launch sync", "campaign management", "launch brief", "sales campaign", "launch dashboard", "campaign close", "wrap report", "launch archive", "selling days", "pacing", "rep leaderboard"

---

## Context: What is Wild Success?

Wild Success is an Australian coaching company (HQ Perth, GMT+8; Operations Brisbane, GMT+10) that runs periodic "launch campaigns" selling coaching programmes. Each launch runs 2-4 weeks with a defined selling period. Sales are tracked in GoHighLevel (GHL) pipelines, synced to Supabase (PostgreSQL), and displayed on dashboards.

**Programmes offered (may vary per launch):**
- Business Coaching (~$7,800 AUD)
- Masters of Coaching / MOC (~$7,000 AUD)
- Human Design (~$6,700 AUD)
- AI (~$6,800 AUD)
- Astrology (~$4,700 AUD)
- Wellness (~$4,600 AUD)
- Breathwork, Meditation, SMMC, Personal Branding (may appear in some launches)

**Key people:**
- Jaydyn Rosevear — Head of Operations (Brisbane GMT+10, WhatsApp preferred)
- Calvin Coyles — CEO/Founder (Perth GMT+8)
- Kim Barrett — Head of Sales & Marketing
- ~10 sales reps per launch

---

## Phase 1: Pre-Launch Setup

### 1.1 Gather Launch Parameters from Exec Team

Before any technical setup, confirm ALL of the following with the exec team (typically Jaydyn):

| Parameter | Example (L3) | Notes |
|-----------|-------------|-------|
| Launch name / ID | L3 | Sequential: L1, L2, L3, L4... |
| Selling start date | 2026-03-24 | First day reps can close deals |
| Selling end date | 2026-04-12 | Last day of campaign |
| Total selling days | 20 | Count of calendar days inclusive |
| Revenue target (AUD) | $2,500,000 | Overall campaign target |
| Sales target | 379 | Total number of sales targeted |
| PIF target | 76 | Paid-in-full target (typically ~20% of sales) |
| Programmes included | Business, MOC, HD, AI, Astrology, Wellness | Which programmes are being sold this launch |
| Rep roster | Daniel Lewis, Anne Breakwell, Mandy Foxton, etc. | Full list of active reps |
| Pipeline IDs per programme | See §1.2 | From GHL — one pipeline per programme per launch |

**Do NOT proceed without all parameters. Missing data = wrong dashboards.**

### 1.2 Identify GHL Pipelines

Each programme in a launch has its own pipeline in its GHL sub-account. You need the pipeline ID for each.

**GHL Sub-Account Location IDs:**

| Sub-Account | Location ID |
|-------------|------------|
| Wild Success (main / "WILD Central") | `POOwEAaRUV4mWkn72yBa` |
| Human Design | `7uhGOEE0XXWrxiXHPv0o` |
| Breathwork | `OAhLgH9zYEVCYYRuRpCN` |
| AI | `zFOxEl40KGPj5B7ETDJM` |
| SMMC | `Xebha2i71m3aEkhNYPAh` |
| Personal Branding | `z33PQnGmPgH7xyrNqnBd` |
| Meditation | `GmLZzsWRaH0s31XCwKPR` |
| Astrology | (confirm with Jaydyn) |
| Biohacking | (confirm with Jaydyn) |

**To list pipelines in a sub-account:**

```bash
curl -s "https://services.leadconnectorhq.com/opportunities/pipelines" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" | jq '.pipelines[] | {id, name}'
```

Each pipeline has stages. You need a **local stage map** for each pipeline (see §1.3).

### 1.3 Build the Local Stage Map

For each pipeline, list ALL stages and categorise them:

```bash
curl -s "https://services.leadconnectorhq.com/opportunities/pipelines/<PIPELINE_ID>" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" | jq '.stages[] | {id, name}'
```

Create a stage map object like:

```json
{
  "pipeline_id": "abc123",
  "programme": "Business Coaching",
  "stages": {
    "stage_id_1": {"name": "New Lead", "category": "open"},
    "stage_id_2": {"name": "Booked", "category": "open"},
    "stage_id_3": {"name": "Showed", "category": "open"},
    "stage_id_4": {"name": "Not Yet Called", "category": "open"},
    "stage_id_5": {"name": "Won", "category": "won"},
    "stage_id_6": {"name": "PIF", "category": "won"},
    "stage_id_7": {"name": "Payment Plan", "category": "won"},
    "stage_id_8": {"name": "Future PP", "category": "won"},
    "stage_id_9": {"name": "Lost", "category": "lost"},
    "stage_id_10": {"name": "RTC", "category": "lost"}
  }
}
```

Categories:
- **won** = counted as a sale (Won, PIF, Payment Plan, Future PP)
- **open** = in pipeline but not yet closed
- **lost** = not a sale (Lost, RTC / Return to Community)

> ⚠️ **WARNING — REGRESSION T-11: Missing Stage Map Entries**
> If a stage exists in GHL but is NOT in your local stage map, opportunities in that stage are **silently dropped** from all counts. They simply don't appear in sales, revenue, or any metric.
>
> **Real example:** During L3, a "Future PP" stage was added to all 6 GHL pipelines mid-campaign. It was missing from the local stage maps. Result: 6 Wellness sales (Caroline Hayner's) and wins across other programmes were silently excluded. Revenue was understated. The rep leaderboard was wrong.
>
> **Fix:** After ANY pipeline stage changes in GHL, immediately update the local stage map. Run a verification: count opportunities per stage in GHL vs. your local categorisation. Any mismatch = missing stage.

### 1.4 Define PIF Detection Rules

> ⚠️ **WARNING — PIF TAG FIX (Critical Regression)**
> Do NOT detect PIFs by dollar threshold (e.g., monetary_value >= $3,000). This was the original approach and it produced wrong numbers ($400+ discrepancy in L3).
>
> **Correct method:** Look up the contact's tags in GHL. Each programme has specific PIF/PP tags:
> - Tags containing "pif" → Paid in Full (e.g., `advprac | pif | $2000`, `biz coaching | pif | $7800`)
> - Tags containing "pp" → Payment Plan (e.g., `advprac | pp | $50`, `biz coaching | pp | $250`)
> - No matching tag → default to Payment Plan
>
> **Why this matters:** Some PIFs have monetary_value that looks like a PP (e.g., scholarship PIFs at $2,000). Some PPs have high monetary_value that looks like PIF. Only the tag is the source of truth.

**To look up contact tags:**

```bash
curl -s "https://services.leadconnectorhq.com/contacts/<CONTACT_ID>" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" | jq '.contact.tags'
```

### 1.5 Set Up Supabase Tables

Ensure these tables exist in your Supabase project:

```sql
-- Opportunities table (main sync target)
CREATE TABLE IF NOT EXISTS opportunities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  opportunity_id TEXT UNIQUE NOT NULL,  -- GHL opportunity ID
  contact_id TEXT,
  contact_name TEXT,
  contact_email TEXT,
  pipeline_id TEXT,
  pipeline_name TEXT,
  stage_id TEXT,
  stage_name TEXT,
  stage_category TEXT,  -- 'won', 'open', 'lost'
  programme TEXT,
  launch TEXT,  -- 'L3', 'L4', etc.
  monetary_value NUMERIC,
  currency TEXT DEFAULT 'AUD',
  assigned_to TEXT,  -- rep name
  is_pif BOOLEAN DEFAULT FALSE,
  pif_source TEXT,  -- 'tag' or 'threshold' (should always be 'tag')
  source TEXT,
  status TEXT,
  location_id TEXT,
  created_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ,
  synced_at TIMESTAMPTZ DEFAULT NOW()
);

-- Daily progression snapshots
CREATE TABLE IF NOT EXISTS daily_progression (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  launch TEXT NOT NULL,
  date DATE NOT NULL,
  day_number INTEGER,
  total_sales INTEGER,
  total_revenue NUMERIC,
  total_pifs INTEGER,
  total_opps INTEGER,
  programme_breakdown JSONB,  -- {"Business": {"sales": 10, "revenue": 78000}, ...}
  rep_breakdown JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(launch, date)
);

-- Launch archives (campaign summaries)
CREATE TABLE IF NOT EXISTS launch_archives (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  launch TEXT UNIQUE NOT NULL,
  start_date DATE,
  end_date DATE,
  selling_days INTEGER,
  target_revenue NUMERIC,
  target_sales INTEGER,
  target_pifs INTEGER,
  actual_revenue NUMERIC,
  actual_sales INTEGER,
  actual_pifs INTEGER,
  actual_opps INTEGER,
  achievement_pct NUMERIC,
  programmes JSONB,
  rep_leaderboard JSONB,
  daily_progression JSONB,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 1.6 Set Up Rep Roster

Create a mapping of rep names (as they appear in GHL `assigned_to` field) to normalised names:

```json
{
  "Daniel Lewis": "Daniel Lewis",
  "daniel lewis": "Daniel Lewis",
  "Anne Breakwell": "Anne Breakwell",
  "Mandy Foxton": "Mandy Foxton",
  "Gil Nelson": "Gil Nelson",
  "Chrissy Lehpamer": "Chrissy Lehpamer",
  "Lisa Pace-Renata": "Lisa Pace-Renata",
  "Jacob Weatherley": "Jacob Weatherley",
  "Elle Porter": "Elle Porter",
  "Ahmed Osman": "Ahmed Osman",
  "Caroline Hayner": "Caroline Hayner",
  "Calvin Coyles": "Calvin Coyles"
}
```

GHL is inconsistent with casing. Always normalise.

---

## Phase 2: Daily Sync (During Active Campaign)

### 2.1 Pulling Opportunities from GHL

For each programme pipeline, pull all opportunities:

```bash
curl -s "https://services.leadconnectorhq.com/opportunities/search" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d '{
    "location_id": "<LOCATION_ID>",
    "pipeline_id": "<PIPELINE_ID>",
    "limit": 100
  }'
```

> ⚠️ **WARNING — CRITICAL PAGINATION GOTCHA**
> GHL pagination requires BOTH `startAfterId` AND `startAfter` parameters. Sending only one returns page 1 infinitely — it looks like you're paginating but you're stuck.
>
> ```json
> {
>   "location_id": "<LOCATION_ID>",
>   "pipeline_id": "<PIPELINE_ID>",
>   "limit": 100,
>   "startAfterId": "<last_opportunity_id>",
>   "startAfter": "<last_opportunity_updated_at_timestamp>"
> }
> ```
>
> - `startAfterId` = the `id` field of the last opportunity on the previous page
> - `startAfter` = the `updatedAt` (or similar timestamp) of that same opportunity
> - `meta.nextPageUrl` in the response is UNRELIABLE — do not depend on it
>
> **Verification:** Log the first opportunity ID of each page. If it repeats, pagination is broken.

**Complete pagination loop (pseudocode):**

```python
all_opportunities = []
start_after_id = None
start_after = None

while True:
    payload = {
        "location_id": LOCATION_ID,
        "pipeline_id": PIPELINE_ID,
        "limit": 100
    }
    if start_after_id:
        payload["startAfterId"] = start_after_id
        payload["startAfter"] = start_after
    
    response = requests.post(
        "https://services.leadconnectorhq.com/opportunities/search",
        headers={
            "Authorization": f"Bearer {PIT_TOKEN}",
            "Version": "2021-07-28",
            "Content-Type": "application/json"
        },
        json=payload
    )
    
    # Rate limit: 100 req/min — add delay
    time.sleep(0.7)  # ~85 req/min with overhead
    
    data = response.json()
    opportunities = data.get("opportunities", [])
    
    if not opportunities:
        break
    
    all_opportunities.extend(opportunities)
    
    last_opp = opportunities[-1]
    start_after_id = last_opp["id"]
    start_after = last_opp.get("updatedAt", last_opp.get("lastUpdatedAt"))
    
    # Safety: verify we advanced
    if len(opportunities) < 100:
        break  # Last page
```

### 2.2 Rate Limiting

GHL allows ~100 requests per minute per location (sub-account).

- Add `time.sleep(0.7)` between requests (gives ~85 req/min with overhead)
- On HTTP 429: back off exponentially (1s, 2s, 4s, 8s... cap at 60s)
- Add random jitter (0-1s) to prevent thundering herd across multiple sub-accounts

```python
import time, random

def rate_limited_request(url, headers, json_payload, max_retries=5):
    for attempt in range(max_retries):
        response = requests.post(url, headers=headers, json=json_payload)
        if response.status_code == 429:
            wait = min(60, (2 ** attempt) + random.uniform(0, 1))
            time.sleep(wait)
            continue
        response.raise_for_status()
        return response.json()
    raise Exception(f"Max retries exceeded for {url}")
```

### 2.3 Processing Opportunities

For each opportunity returned:

1. **Look up stage** in local stage map → get `stage_category` (won/open/lost)
2. **If stage not in map** → LOG A WARNING and flag immediately (regression T-11)
3. **If won** → look up contact tags for PIF detection (see §1.4)
4. **Extract rep name** from `assignedTo` or `contact.assignedTo` field → normalise using rep roster
5. **Extract revenue** from `monetaryValue` field (in cents or dollars — verify which; GHL is inconsistent)

### 2.4 Revenue Calculation

```python
# GHL monetary_value may be in dollars (not cents) — verify per pipeline
revenue = opportunity.get("monetaryValue", 0)
if revenue and isinstance(revenue, (int, float)):
    # If value looks like cents (> 100,000 for a $1,000 sale), divide by 100
    # Wild Success programmes range $2,000-$8,000 AUD
    if revenue > 50000:
        revenue = revenue / 100
```

**Currency:** All revenue is AUD unless explicitly stated otherwise. USD conversion uses ~0.667 rate (confirm current rate).

### 2.5 Upserting to Supabase

```python
# Using supabase-py or raw SQL
upsert_query = """
INSERT INTO opportunities (
    opportunity_id, contact_id, contact_name, contact_email,
    pipeline_id, pipeline_name, stage_id, stage_name, stage_category,
    programme, launch, monetary_value, currency, assigned_to,
    is_pif, pif_source, location_id, created_at, updated_at, synced_at
) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, NOW())
ON CONFLICT (opportunity_id) DO UPDATE SET
    stage_id = EXCLUDED.stage_id,
    stage_name = EXCLUDED.stage_name,
    stage_category = EXCLUDED.stage_category,
    monetary_value = EXCLUDED.monetary_value,
    assigned_to = EXCLUDED.assigned_to,
    is_pif = EXCLUDED.is_pif,
    updated_at = EXCLUDED.updated_at,
    synced_at = NOW()
"""
```

**Always `conn.commit()` after writes.** PostgreSQL silently rolls back uncommitted transactions on connection close. No errors, no warnings — data just disappears.

### 2.6 Daily Progression Snapshot

After each sync, calculate and store a daily snapshot:

```python
snapshot = {
    "launch": "L3",
    "date": "2026-04-05",
    "day_number": 13,  # days since selling_start
    "total_sales": count_where(stage_category='won'),
    "total_revenue": sum_where(stage_category='won', field='monetary_value'),
    "total_pifs": count_where(is_pif=True, stage_category='won'),
    "total_opps": count_all(),
    "programme_breakdown": {
        "Business Coaching": {"sales": 93, "revenue": 735900, "pifs": 10},
        "Masters of Coaching": {"sales": 78, "revenue": 550200, "pifs": 17},
        # ...
    },
    "rep_breakdown": {
        "Daniel Lewis": {"sales": 29, "revenue": 221100, "pifs": 3},
        # ...
    }
}
```

Insert with `ON CONFLICT (launch, date) DO UPDATE` to allow re-syncs on the same day.

---

## Phase 3: Daily Brief (During Active Campaign)

### 3.1 Brief Format

Deliver every morning before Jaydyn starts his day (Brisbane GMT+10, ideally 6-7 AM AEST).

**Template:**

```
📊 [LAUNCH] Day [N] Sales Brief — [DATE]

💰 Revenue: $[X] AUD ([Y]% of $[TARGET])
📈 Sales: [X] ([Y]% of [TARGET])
💎 PIFs: [X] ([Y]% of [TARGET])
📋 Opps: [X]

📏 Pacing: [AHEAD/BEHIND/ON TRACK]
Expected by today: $[X] AUD / [Y] sales
Actual: $[X] AUD / [Y] sales
Gap: [+/-$X] AUD / [+/-Y] sales

🔝 Yesterday's Top Closers:
• [Rep 1] — [X] sales ($[Y])
• [Rep 2] — [X] sales ($[Y])

📊 Programme Breakdown:
• Business: [X] sales / $[Y]
• MOC: [X] sales / $[Y]
• HD: [X] sales / $[Y]
[etc.]

🚩 Flags:
• [Any data issues, unusual patterns, sync failures]

💡 Insight:
• [One actionable observation — e.g., "AI conversion still lowest at 25%. Consider dedicated closer."]
```

### 3.2 Pacing Calculation

```python
# Linear pacing model
target_revenue = 2500000  # AUD
total_selling_days = 20
days_elapsed = (today - selling_start).days + 1  # 1-indexed

expected_revenue = target_revenue * (days_elapsed / total_selling_days)
expected_sales = target_sales * (days_elapsed / total_selling_days)

actual_revenue = query_total_revenue()
actual_sales = query_total_sales()

# Pacing status
if actual_revenue >= expected_revenue * 1.05:
    pacing = "AHEAD ✅"
elif actual_revenue >= expected_revenue * 0.95:
    pacing = "ON TRACK ➡️"
else:
    pacing = "BEHIND ⚠️"
```

> ⚠️ **WARNING — Pacing Bug (from L3)**
> The pacing badge must compare against the expected value **for today's date**, not against the final target. Comparing actual vs. total target will always show "Behind" until the last day. The correct formula is:
> `expected_for_today = target × (days_elapsed / total_selling_days)`

### 3.3 Delivery

- **Primary:** WhatsApp to Jaydyn (+61456183609)
- **Secondary:** Slack #sales-chat or #ops channel for team visibility
- **Voice note option:** Keep TTS version under 1,500 characters. Strip emojis and markdown for TTS.

### 3.4 Rep Leaderboard

Track per-rep performance across all programmes:

| Rank | Rep | Sales | Revenue AUD | PIFs | Programmes |
|------|-----|-------|-------------|------|------------|
| 1 | Mandy Foxton | 37 | $268,800 | 5 | MOC, HD, AI, Biz, Well |
| 2 | Anne Breakwell | 32 | $224,400 | 9 | MOC, HD, Biz |
| 3 | Gil Nelson | 32 | $217,200 | 7 | MOC, HD, AI, Biz, Astro |

Include in the daily brief when significant changes occur (new #1, milestone reached).

---

## Phase 4: Campaign Close-Out

### 4.1 Final Sync

On the last selling day (or day after), run a final sync:

1. Pull ALL opportunities from ALL programme pipelines one last time
2. Verify counts match GHL (spot-check 2-3 pipelines manually)
3. Run PIF tag detection on all won opportunities
4. Store final daily progression snapshot
5. Note the exact timestamp of the final sync

### 4.2 Disable All Campaign Crons

Disable every cron job related to this launch:
- Dashboard sync cron
- Daily sales brief cron
- RTC report cron (if applicable)
- Any other launch-specific automations

**Do this immediately after final sync. Stale crons posting old data = confusion.**

### 4.3 Archive to Supabase

Insert a summary row into `launch_archives`:

```python
archive = {
    "launch": "L3",
    "start_date": "2026-03-24",
    "end_date": "2026-04-12",
    "selling_days": 20,
    "target_revenue": 2500000,
    "target_sales": 379,
    "target_pifs": 76,
    "actual_revenue": 2151000,
    "actual_sales": 315,
    "actual_pifs": 94,
    "actual_opps": 1374,
    "achievement_pct": 86.0,
    "programmes": {/* full programme breakdown */},
    "rep_leaderboard": {/* full rep data */},
    "daily_progression": {/* array of daily snapshots */},
    "notes": "PIF target exceeded (123.7%). Revenue 86% of target. AI conversion weakest programme."
}
```

### 4.4 Generate Wrap Report

The wrap report should include:

1. **Headline numbers:** Total sales, revenue, PIFs, opps, achievement %
2. **Programme breakdown:** Per-programme sales, revenue, PIFs, PIF%, opps, show rate, conversion rate
3. **Rep leaderboard:** Top 10+ reps with sales, revenue, PIFs, programmes covered
4. **Daily sales curve:** Day-by-day progression with milestones noted
5. **Target comparison:** This launch vs. target, and vs. previous launch
6. **Trends & patterns:** What worked, what needs improvement, recommendations for next launch
7. **Operational notes:** What crons were disabled, where data is archived

### 4.5 L3 Example (Real Numbers)

**Use these as a reference for what "good" looks like:**

| Metric | L3 Final | L3 Target | Achievement |
|--------|----------|-----------|-------------|
| Total Sales | 315 | 379 | 83.1% |
| Total Revenue (AUD) | $2,151,000 | $2,500,000 | 86.0% |
| Total Revenue (USD) | $1,434,000 | $1,666,667 | 86.0% |
| Paid in Full (PIFs) | 94 | 76 | 123.7% ✅ |
| Total Opportunities | 1,374 | 1,360 | 101.0% ✅ |
| PIF % | 29.8% | 20% | Exceeded |
| Selling Days | 20 | 20 | — |
| Revenue/Day | $107,550 | $125,000 | 86.0% |
| Sales/Day | 15.75 | 18.95 | 83.1% |

**L3 Top Reps:**
1. Mandy Foxton — 37 sales, $268,800, 5 programmes
2. Anne Breakwell — 32 sales, $224,400
3. Gil Nelson — 32 sales, $217,200
4. Chrissy Lehpamer — 31 sales, $139,800 (22 PIFs — 71% PIF rate)
5. Daniel Lewis — 29 sales, $221,100

---

## Phase 5: Cron Setup

### Recommended Cron Schedule During Active Campaign

| Cron | Frequency | Description |
|------|-----------|-------------|
| Pipeline sync | Every 60 min (6am-11pm AEST) | Pull all opps, upsert to Supabase |
| Daily brief | 6:00 AM AEST daily | Generate and deliver morning brief |
| Rep leaderboard | Every 60 min (with sync) | Update rep standings |
| Daily snapshot | 11:00 PM AEST daily | Freeze daily progression |
| RTC report | 9:00 AM AEST daily | Report return-to-community (optional) |

### Between Campaigns

- All launch-specific crons DISABLED
- System health checks continue (Supabase connectivity, GHL auth validity)

---

## Known Regressions & Gotchas

| ID | Description | Impact | Fix |
|----|-------------|--------|-----|
| T-11 | Missing stage in local stage map → sales silently dropped | 6+ sales invisible in L3 | Always verify ALL stages are mapped after any GHL change |
| PIF-01 | Dollar threshold PIF detection → wrong PIF counts | $400+ discrepancy | Use contact tags, never dollar amounts |
| PAG-01 | GHL pagination with only one cursor param → stuck on page 1 | Incomplete data | Always send BOTH startAfterId AND startAfter |
| PACE-01 | Pacing compared to final target, not today's expected | Always shows "Behind" | Compare to linear interpolation for today's date |
| DB-01 | Missing conn.commit() → data silently lost | Zero rows written | Always explicitly commit after writes |

---

## Checklist: New Launch Setup

```
[ ] Launch parameters confirmed (dates, targets, programmes, reps)
[ ] Pipeline IDs collected for all programmes
[ ] Stage maps built and verified (ALL stages present)
[ ] PIF tag patterns documented per programme
[ ] Supabase tables verified / created
[ ] Rep roster mapping created
[ ] Sync script tested with one pipeline
[ ] Full sync run and verified against GHL counts
[ ] Daily brief template customised for this launch
[ ] Cron jobs created and tested
[ ] Dashboard configured for new launch
[ ] Jaydyn confirms everything looks right
```

---

*This skill is self-contained. All API endpoints, field names, table schemas, and known regressions are documented above. A fresh agent on a new machine should be able to execute a complete launch campaign using only this document.*
