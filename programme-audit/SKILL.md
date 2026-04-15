# Programme Audit

**Version:** 1.0.0
**Description:** Audit a Wild Success coaching programme for session attendance, sales revenue, PIF/PP split, compliance (consecutive attendance), graduation rates, and cross-account sales. Uses the corrected methodology established after Atlas reviewed the Meditation February 2026 audit — including compound tag filters, cross-account Business Coaching lookups, tag-based PIF detection, and consecutive compliance chains.

**Trigger phrases:** "programme audit", "program audit", "audit meditation", "audit breathwork", "attendance audit", "compliance audit", "graduation audit", "sales audit", "programme review", "session attendance", "programme report"

---

## Context: What is a Programme Audit?

Wild Success runs free multi-session coaching programmes (typically 4-8 sessions over 2-4 weeks). During and after each programme, an audit measures:

1. **Registration** — how many people signed up
2. **Session attendance** — how many registered people attended each session (live or replay)
3. **Compliance** — consecutive attendance from Session 1 through Session N (no gaps)
4. **Graduation** — how many completed the programme
5. **Sales** — products/upgrades sold during the programme (Advanced Practitioner, Business Coaching, etc.)
6. **PIF/PP split** — how many paid in full vs. payment plans
7. **Territory** — geographic breakdown of registrants

**Why audits matter:** They measure programme health, inform marketing ROI, identify retention patterns, and feed into launch wrap reports.

---

## Critical Rules (Non-Negotiable)

These four rules were established after the Meditation February 2026 audit was corrected by Atlas. Every future audit MUST follow them.

### Rule 1: Session Attendance — Compound Filter (NOT Independent Tag Counts)

> ⚠️ **WARNING — Independent tag counting overcounts by ~37%**
>
> **WRONG approach:** Count contacts with `meditfeb26-1 attended` tag → 1,350. Count contacts with `meditfeb26-1 replay` tag → 420. Total attendance = 1,770.
>
> **Why it's wrong:** A contact can have BOTH the `attended` and `replay` tags. You double-count them. Additionally, non-registered people may have attendance tags (from viewing links, tag automation errors, etc.).
>
> **CORRECT approach:** Use a COMPOUND filter requiring BOTH the registration tag AND the session tag.

**API call for correct attendance count:**

```bash
curl -s "https://services.leadconnectorhq.com/contacts/search" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d '{
    "locationId": "<LOCATION_ID>",
    "filters": [
      {
        "field": "tags",
        "operator": "contains",
        "value": "meditation february 2026"
      },
      {
        "field": "tags",
        "operator": "contains",
        "value": ["meditfeb26-1 attended", "meditfeb26-1 replay"]
      }
    ],
    "limit": 1
  }'
```

**How the filter logic works:**
- The first filter (`"value": "meditation february 2026"`) requires the contact to be REGISTERED
- The second filter (`"value": ["meditfeb26-1 attended", "meditfeb26-1 replay"]`) uses an array = OR logic: attended OR replay
- Two filters together = AND logic: must be registered AND (attended OR replay)
- The response `meta.total` gives you the count without fetching all records (`limit: 1`)

**Tag naming conventions:**

| Component | Pattern | Example |
|-----------|---------|---------|
| Registration tag | `[programme] [month] [year]` | `meditation february 2026` |
| Session attendance (live) | `[prefix]-[N] attended` | `meditfeb26-1 attended` |
| Session replay | `[prefix]-[N] replay` | `meditfeb26-1 replay` |
| Graduation tag | `[programme] graduate` or `[prefix] graduate` | `meditation graduate` |

**Prefixes vary by programme:**
- Meditation → `meditfeb26`
- NLP/FT → `nlpftmar26`
- Human Design → `hdfeb26`
- AI → `aifeb26`
- Breathwork → `breathfeb26`

Always list ALL tags in the sub-account first to confirm exact tag names:

```bash
# List all tags in a location (paginated)
curl -s "https://services.leadconnectorhq.com/locations/<LOCATION_ID>/tags" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28"
```

### Rule 2: Cross-Account Sales — ALWAYS Check WILD Central

> ⚠️ **WARNING — Missing cross-account sales understates revenue by $20K-$30K+**
>
> **Business Coaching is pitched across ALL programmes** but sales always live in the WILD Central sub-account (Location ID: `POOwEAaRUV4mWkn72yBa`), NOT in the programme's own sub-account.
>
> **Real example:** Meditation Feb 2026 audit initially showed $0 Business Coaching sales because only the Meditation sub-account was checked. The actual number was $29,100 AUD (4 sales) — all in WILD Central.

**How to find cross-account Business Coaching sales:**

1. In the **programme's sub-account**, find contacts with the business application tag:
   ```
   Tag: "[programme] - business application"
   Example: "meditation - business application"
   ```

2. Get those contact IDs

3. In **WILD Central** (POOwEAaRUV4mWkn72yBa), search the Business Coaching pipeline for opportunities matching those contact IDs:
   ```bash
   # Find Business Coaching pipeline in WILD Central
   curl -s "https://services.leadconnectorhq.com/opportunities/pipelines?locationId=POOwEAaRUV4mWkn72yBa" \
     -H "Authorization: Bearer <WILD_CENTRAL_PIT>" \
     -H "Version: 2021-07-28" | jq '.pipelines[] | select(.name | contains("Business"))'
   ```

4. Cross-reference contact IDs with won opportunities in that pipeline

**This applies to EVERY programme audit. Not just meditation. Always check WILD Central.**

### Rule 3: PIF/PP Detection — Use Contact Tags, NOT Dollar Threshold

> ⚠️ **WARNING — Dollar threshold was $400 off in the Meditation audit**
>
> **WRONG:** `monetaryValue <= $3,000` = PIF → Reported 23 PIFs
> **CORRECT:** Tag lookup → Reported 25 PIFs (21 normal + 4 Founding Hero)
>
> The $400 discrepancy came from scholarship PIFs that had values below the threshold and payment plans with high initial values above it.

**Correct method:**

```python
def classify_pif_pp(contact_tags, programme_prefix):
    """
    Classify a contact as PIF or PP based on their tags.
    
    Tags containing 'pif' → Paid in Full
    Tags containing 'pp' with pipe separator → Payment Plan
    No matching tag → Default to PP
    """
    for tag in contact_tags:
        tag_lower = tag.lower()
        if "pif" in tag_lower:
            return "PIF"
        # Check for PP tags (format: "advprac | pp | $50")
        if "| pp |" in tag_lower or "| pp" in tag_lower:
            return "PP"
    return "PP"  # Default if no tag found
```

**Founding Hero (FH) tags:** Some contacts have "FH" or "founding hero" tags. These count as PIFs with special pricing. Include them in the PIF count but note them separately.

### Rule 4: Compliance — Consecutive Attendance (AND Chain)

> ⚠️ **WARNING — Compliance was completely MISSING from the first Meditation audit**
>
> Compliance is not the same as total attendance. It measures **consecutive attendance from Session 1 through Session N with NO gaps**.

**Compliance definitions:**

| Metric | Filter | Meaning |
|--------|--------|---------|
| S1 compliance | reg AND s1 | Attended at least Session 1 |
| S1-2 compliance | reg AND s1 AND s2 | Attended Sessions 1 AND 2 consecutively |
| S1-3 compliance | reg AND s1 AND s2 AND s3 | Attended Sessions 1, 2, AND 3 consecutively |
| S1-8 compliance | reg AND s1 AND s2 AND s3 AND s4 AND s5 AND s6 AND s7 AND s8 | Attended ALL 8 sessions |

**API call for compliance (e.g., S1-3):**

```bash
curl -s "https://services.leadconnectorhq.com/contacts/search" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d '{
    "locationId": "<LOCATION_ID>",
    "filters": [
      {"field": "tags", "operator": "contains", "value": "meditation february 2026"},
      {"field": "tags", "operator": "contains", "value": ["meditfeb26-1 attended", "meditfeb26-1 replay"]},
      {"field": "tags", "operator": "contains", "value": ["meditfeb26-2 attended", "meditfeb26-2 replay"]},
      {"field": "tags", "operator": "contains", "value": ["meditfeb26-3 attended", "meditfeb26-3 replay"]}
    ],
    "limit": 1
  }'
```

Each filter is ANDed together. Within each session filter, the array provides OR (attended OR replay). The result: contacts who were registered AND attended (live or replay) Sessions 1, 2, AND 3.

**Compliance shows the retention funnel.** A healthy programme has gradual drop-off. Steep drops between specific sessions indicate content or scheduling problems.

---

## Step-by-Step Audit Process

### Step 1: Identify the Programme

Gather:
- Programme name (e.g., "Meditation February 2026")
- Sub-account location ID (see table below)
- PIT token for that sub-account
- Number of sessions (typically 4 or 8)
- Tag prefix (e.g., `meditfeb26`)
- Registration tag (e.g., `meditation february 2026`)
- Whether the programme sells upgrades (Advanced Practitioner, Business Coaching, etc.)

**Sub-account reference:**

| Programme | Location ID | Notes |
|-----------|------------|-------|
| Wild Success (WILD Central) | `POOwEAaRUV4mWkn72yBa` | Business Coaching pipeline lives here |
| Human Design | `7uhGOEE0XXWrxiXHPv0o` | |
| Breathwork | `OAhLgH9zYEVCYYRuRpCN` | |
| AI | `zFOxEl40KGPj5B7ETDJM` | |
| SMMC | `Xebha2i71m3aEkhNYPAh` | |
| Personal Branding | `z33PQnGmPgH7xyrNqnBd` | |
| Meditation | `GmLZzsWRaH0s31XCwKPR` | |

### Step 2: List All Tags

Before any counting, list all tags to confirm exact naming:

```bash
curl -s "https://services.leadconnectorhq.com/locations/<LOCATION_ID>/tags" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" | jq '.tags[] | select(.name | test("meditfeb26|meditation"))'
```

Document every relevant tag: registration, session attendance (live + replay), graduation, sales tags (pif, pp), and business application tags.

### Step 3: Registration Count

```bash
curl -s "https://services.leadconnectorhq.com/contacts/search" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d '{
    "locationId": "<LOCATION_ID>",
    "filters": [
      {"field": "tags", "operator": "contains", "value": "meditation february 2026"}
    ],
    "limit": 1
  }'
```

`meta.total` = registration count.

### Step 4: Session Attendance (Per Session)

For each session (1 through N), use the compound filter from Rule 1:

```bash
# Session 1
curl -s ... -d '{
    "locationId": "...",
    "filters": [
      {"field": "tags", "operator": "contains", "value": "meditation february 2026"},
      {"field": "tags", "operator": "contains", "value": ["meditfeb26-1 attended", "meditfeb26-1 replay"]}
    ],
    "limit": 1
  }'

# Session 2
curl -s ... -d '{
    "locationId": "...",
    "filters": [
      {"field": "tags", "operator": "contains", "value": "meditation february 2026"},
      {"field": "tags", "operator": "contains", "value": ["meditfeb26-2 attended", "meditfeb26-2 replay"]}
    ],
    "limit": 1
  }'

# ... repeat for each session
```

Record `meta.total` for each.

### Step 5: Compliance (Consecutive Attendance)

Build progressive AND chains as described in Rule 4. For an 8-session programme, you need 8 queries:

| Query | Filters | Gives You |
|-------|---------|-----------|
| S1 | reg AND s1 | S1 compliance |
| S1-2 | reg AND s1 AND s2 | S1-2 compliance |
| S1-3 | reg AND s1 AND s2 AND s3 | S1-3 compliance |
| ... | ... | ... |
| S1-8 | reg AND s1 AND s2 AND ... AND s8 | Full compliance |

### Step 6: Graduation Count

```bash
curl -s "https://services.leadconnectorhq.com/contacts/search" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d '{
    "locationId": "<LOCATION_ID>",
    "filters": [
      {"field": "tags", "operator": "contains", "value": "meditation february 2026"},
      {"field": "tags", "operator": "contains", "value": "meditation graduate"}
    ],
    "limit": 1
  }'
```

### Step 7: Pipeline Sales (Sub-Account)

List pipelines in the sub-account, then count won opportunities:

```bash
# List pipelines
curl -s "https://services.leadconnectorhq.com/opportunities/pipelines?locationId=<LOCATION_ID>" \
  -H "Authorization: Bearer <PIT_TOKEN>" \
  -H "Version: 2021-07-28"

# Search won opportunities in the relevant pipeline
# Use opportunity search with stage filtering
```

For each won opportunity:
- Record the `monetaryValue` (revenue)
- Look up contact tags for PIF/PP classification (Rule 3)

### Step 8: Cross-Account Business Coaching Sales (WILD Central)

**This step is MANDATORY for every programme audit.**

1. In the programme's sub-account, find contacts with the business application tag
2. Get their contact IDs
3. In WILD Central (`POOwEAaRUV4mWkn72yBa`), search the Business Coaching pipeline for matching opportunities

See Rule 2 for detailed instructions.

### Step 9: PIF/PP Split

For every won opportunity (from Steps 7 and 8):
- Look up the contact's tags
- Classify as PIF or PP (Rule 3)
- Calculate PIF cash total and PP cash total separately

```
PIF cash = sum of monetary_value for PIF contacts
PP cash = sum of monetary_value for PP contacts
```

### Step 10: Territory / Geographic Breakdown

Pull registered contacts and group by country:

```bash
# Search with registration tag, but this time fetch contacts (higher limit)
# Extract contact.country or contact.address.country
```

Standard territory grouping:
- **APAC** (Australia, New Zealand, SE Asia, India)
- **Americas** (USA, Canada, Latin America)
- **UK/Euro** (UK, Ireland, Europe)

### Step 11: Compile the Report

---

## Output Format: Programme Audit Report

```markdown
# Programme Audit Report: [Programme Name]
## Period: [Start Date] – [End Date]
## Audited: [Today's Date]
## Data Source: GHL API ([Sub-account Name], Location: [ID])

---

### Registration
- **Total Registered:** [N]
- **Registration Tag:** [tag name]

### Session Attendance (Registered Contacts Only)
| Session | Attended/Replay | % of Registered | Δ from Previous |
|---------|-----------------|-----------------|-----------------|
| S1 | [N] | [%] | — |
| S2 | [N] | [%] | [–X] |
| S3 | [N] | [%] | [–X] |
| ... | | | |

### Compliance (Consecutive from S1)
| Metric | Count | % of Registered | Drop from Previous |
|--------|-------|------------------|--------------------|
| S1 | [N] | [%] | — |
| S1-2 | [N] | [%] | [–X] |
| S1-3 | [N] | [%] | [–X] |
| ... | | | |

### Graduation
- **Graduates:** [N] ([%] of registered)

### Sales — [Programme's Sub-Account]
| Product | Sales | Revenue AUD | PIFs | PPs |
|---------|-------|-------------|------|-----|
| [Product 1] | [N] | $[X] | [N] | [N] |
| [Product 2] | [N] | $[X] | [N] | [N] |

### Sales — Business Coaching (WILD Central)
| Sales | Revenue AUD | PIFs | PPs |
|-------|-------------|------|-----|
| [N] | $[X] | [N] | [N] |

### Cash Summary
- **PIF Cash:** $[X] AUD
- **PP Cash:** $[X] AUD
- **Total Cash:** $[X] AUD

### Territory
| Region | Count | % |
|--------|-------|---|
| APAC | [N] | [%] |
| Americas | [N] | [%] |
| UK/Euro | [N] | [%] |

### Notes
- [Any flags, data quality issues, discrepancies found]
```

---

## Reference: Meditation Feb 2026 — Atlas-Verified Numbers

Use these as a benchmark for what "correct" looks like:

| Metric | Value |
|--------|-------|
| Registrations | 2,234 |
| S1 attendance | 993 |
| S2 attendance | 836 |
| S3 attendance | 763 |
| S4 attendance | 698 |
| S5 attendance | 674 |
| S6 attendance | 607 |
| S7 attendance | 539 |
| S8 attendance | 507 |
| S1 compliance | 993 |
| S1-2 compliance | 784 |
| S1-3 compliance | 677 |
| S1-4 compliance | 610 |
| S1-5 compliance | 572 |
| S1-6 compliance | 520 |
| S1-7 compliance | 449 |
| S1-8 compliance | 413 |
| Graduates | 612 |
| Adv Prac Sold | 25 (21 normal + 4 FH) = $121,800 AUD |
| Business Sold (WILD Central) | 4 = $29,100 AUD |
| PIF cash | $36,600 AUD |
| PP cash | $85,200 AUD |
| Territory — APAC | 1,600 |
| Territory — Americas | 765 |
| Territory — UK/Euro | 21 |

**What the initial (incorrect) audit showed vs. corrected:**

| Section | Initial (Wrong) | Corrected | Error |
|---------|----------------|-----------|-------|
| S1 attendance | ~1,360 | 993 | +37% overcount (independent tag counts) |
| Business sales | $0 | $29,100 | Missed cross-account entirely |
| PIF count | 23 (dollar threshold) | 25 (tag lookup) | 2 PIFs miscategorised |
| Compliance | Not calculated | Full chain from S1-S1-8 | Missing section |

---

## General Audit Checklist

```
[ ] Identify programme sub-account + PIT token
[ ] List ALL tags in the sub-account (filter for programme + attendance + sales)
[ ] List ALL pipelines in the sub-account
[ ] Registration count (compound: registration tag only)
[ ] Session attendance per session (compound: reg AND session tag) — Rule 1
[ ] Compliance per session chain (consecutive AND filters) — Rule 4
[ ] Graduation count (compound: reg AND graduate tag)
[ ] Pipeline sales in sub-account (won stages, revenue, rep)
[ ] Cross-account Business Coaching (WILD Central) — Rule 2
[ ] PIF/PP split for all won opportunities (contact tag lookup) — Rule 3
[ ] Territory breakdown (contact country field)
[ ] Compile report in standard format
[ ] Cross-check totals for sanity (e.g., S1 compliance should = S1 attendance)
[ ] Flag any data quality issues
```

---

## Common Pitfalls

| Pitfall | Impact | Prevention |
|---------|--------|------------|
| Counting tags independently | ~37% overcount | Always use compound filter (reg AND session) |
| Only checking programme sub-account for sales | Missing $20K-$30K+ in Business Coaching | Always check WILD Central |
| Using dollar threshold for PIF | $400+ error | Use contact tag lookup |
| Skipping compliance | Missing a key metric | Always build the consecutive AND chain |
| Wrong tag prefix | Zero results | List tags first, confirm exact naming |
| Forgetting replay tags | Undercounting | Always include replay in OR filter with attended |
| Not filtering by registration | Non-registrants included | First filter must always be the registration tag |

---

*This skill is self-contained. A fresh agent can execute a complete programme audit using only this document and GHL API access.*
