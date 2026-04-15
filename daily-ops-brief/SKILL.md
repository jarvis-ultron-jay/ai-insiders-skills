# Daily Ops Brief

**Version:** 1.0.0
**Description:** Generate and deliver a daily operations briefing for the Wild Success executive team. Covers data gathering from GHL (sales), Slack (ops issues), cron health, and active tasks. Includes formatting rules, delivery channels, pacing calculations during active launches, voice note TTS option, and what NOT to include. Designed to be the first thing Jaydyn sees every morning.

**Trigger phrases:** "daily brief", "morning brief", "ops brief", "daily report", "morning report", "daily update", "ops update", "generate brief", "send brief", "morning briefing"

---

## Context: Why the Daily Brief Exists

The Head of Operations (Jaydyn Rosevear, Brisbane GMT+10) needs to start every day with a clear picture of:
- Where the business stands right now (numbers)
- What happened yesterday
- What's on deck today
- What's broken or blocked

**The brief is the #1 daily touchpoint between the AI operations system and the executive team.** If the brief is wrong, trust erodes. If it's late, momentum is lost. If it includes noise, it gets ignored.

---

## Delivery Timing & Channel

### When to Deliver

| Scenario | Time | Notes |
|----------|------|-------|
| Active launch campaign | 6:00 AM AEST (Brisbane GMT+10) | Must include fresh sales data |
| No active launch | 7:00 AM AEST | Can be slightly later |
| Weekend | 8:00 AM AEST (optional) | Brief format only if significant events |

**"Before Jaydyn starts his day"** = before 7:00 AM AEST on weekdays. He checks WhatsApp first thing.

### Where to Deliver

| Channel | Priority | Format | Notes |
|---------|----------|--------|-------|
| **WhatsApp** (Jaydyn) | PRIMARY | Text message | Jaydyn's phone: +61456183609. Keep under 4,000 chars. |
| **Slack** (#ops or #daily-brief) | SECONDARY | Full markdown | Team visibility. Post after WhatsApp. |
| **Voice note** (WhatsApp) | OPTIONAL | TTS audio | Jaydyn loves voice note summaries. See §6. |

> ⚠️ **WARNING — WhatsApp Formatting**
> - NO markdown tables (they render as plain text garbage)
> - NO headers (#, ##) — use **bold** or CAPS for emphasis
> - Bullet lists (•) work fine
> - **Send text FIRST, then any files/attachments as SEPARATE messages.** WhatsApp drops text when files are attached.
> - Keep under 4,000 characters (WhatsApp truncates)

---

## Data Sources

### Source 1: GHL Sales Data (During Active Launch)

Pull from Supabase (synced from GHL) or directly from GHL API:

```sql
-- Total sales, revenue, PIFs for current launch
SELECT
  COUNT(*) FILTER (WHERE stage_category = 'won') AS total_sales,
  SUM(monetary_value) FILTER (WHERE stage_category = 'won') AS total_revenue,
  COUNT(*) FILTER (WHERE is_pif = true AND stage_category = 'won') AS total_pifs,
  COUNT(*) AS total_opps
FROM opportunities
WHERE launch = 'L3';  -- Replace with current launch

-- Yesterday's sales (new wins since yesterday's brief)
SELECT
  COUNT(*) AS new_sales,
  SUM(monetary_value) AS new_revenue,
  assigned_to AS rep,
  programme
FROM opportunities
WHERE launch = 'L3'
  AND stage_category = 'won'
  AND updated_at >= NOW() - INTERVAL '24 hours'
GROUP BY assigned_to, programme
ORDER BY new_sales DESC;
```

If syncing directly from GHL, use the pipeline sync process documented in the `ghl-pipeline-sync` skill.

### Source 2: Slack Activity

Check key Slack channels for anything ops-relevant from the past 24 hours:
- `#ops` — operational issues, team requests
- `#sales-chat` — sales team communications
- `#general` — company-wide announcements
- Any launch-specific channels

Look for: flagged issues, blockers mentioned, team requests, system errors reported.

### Source 3: Cron Health

Check the status of active cron jobs:
- Did all scheduled syncs run successfully?
- Any failures or errors in the last 24 hours?
- Are all expected data pipelines current?

```bash
# Check cron job status (OpenClaw cron system)
openclaw cron list
# Or check specific job logs
```

### Source 4: Active Tasks

Pull from TASKQUEUE.md or active task tracking:
- What's in progress?
- What's blocked?
- What's due today?

### Source 5: Daily Progression (During Active Launch)

```sql
-- Get yesterday's and today's snapshots for delta calculation
SELECT date, total_sales, total_revenue, total_pifs, total_opps
FROM daily_progression
WHERE launch = 'L3'
ORDER BY date DESC
LIMIT 2;
```

---

## Brief Format

### Active Launch Brief (Template)

```
📊 [LAUNCH] Day [N]/[TOTAL] — [DAY OF WEEK] [DATE]

💰 Revenue: $[X] AUD ([Y]% of $[TARGET])
📈 Sales: [X] ([Y]% of [TARGET])
💎 PIFs: [X] ([Y]% of [TARGET])
📋 Opps: [X]

📏 PACING: [STATUS EMOJI + WORD]
• Expected by Day [N]: $[X] / [Y] sales
• Actual: $[X] / [Y] sales
• Gap: [+/-$X] / [+/-Y] sales

📅 YESTERDAY ([+X] sales, $[+Y])
• [Rep 1]: [X] sales ($[Y]) — [programmes]
• [Rep 2]: [X] sales ($[Y]) — [programmes]
• [Rep 3]: [X] sales ($[Y]) — [programmes]
[Only reps who closed yesterday. Skip if no new sales.]

📊 PROGRAMME STANDING
• Business: [X] sales / $[Y] ([Z]% of programme target if set)
• MOC: [X] sales / $[Y]
• HD: [X] sales / $[Y]
• AI: [X] sales / $[Y]
• Astrology: [X] sales / $[Y]
• Wellness: [X] sales / $[Y]

🚩 FLAGS
• [Only genuine issues — data problems, system errors, unusual patterns]
• [Nothing here if nothing to flag]

💡 INSIGHT
• [One actionable observation. Not a platitude. Something useful.]
```

### Non-Launch Brief (Template)

```
☀️ DAILY OPS — [DAY OF WEEK] [DATE]

📌 TODAY'S PRIORITIES
1. [Priority 1 — most important thing]
2. [Priority 2]
3. [Priority 3]

📋 ACTIVE TASKS
• [Task 1] — [status]
• [Task 2] — [status]
• [Task 3] — [status]

🔴 BLOCKED
• [Blocked item] — [why, what's needed]

📢 SLACK HIGHLIGHTS
• [Anything ops-relevant from last 24h]

🛠️ SYSTEM HEALTH
• Crons: [all green / X failures]
• Syncs: [current / stale]
• [Any system issues]
```

---

## Pacing Calculation (Active Launch)

### Linear Pacing Model

```python
# Configuration
target_revenue = 2500000  # AUD — from launch parameters
target_sales = 379
total_selling_days = 20
selling_start = date(2026, 3, 24)  # First selling day

# Calculate
today = date.today()
days_elapsed = (today - selling_start).days + 1  # 1-indexed (Day 1 = selling_start)

# Cap at total selling days
days_elapsed = min(days_elapsed, total_selling_days)

# Expected values for today
expected_revenue = target_revenue * (days_elapsed / total_selling_days)
expected_sales = target_sales * (days_elapsed / total_selling_days)

# Actual values (from Supabase or GHL)
actual_revenue = query_total_revenue()
actual_sales = query_total_sales()

# Revenue pacing
revenue_gap = actual_revenue - expected_revenue
revenue_gap_pct = (actual_revenue / expected_revenue - 1) * 100 if expected_revenue > 0 else 0

# Sales pacing
sales_gap = actual_sales - expected_sales
sales_gap_pct = (actual_sales / expected_sales - 1) * 100 if expected_sales > 0 else 0
```

### Pacing Status

| Condition | Status | Emoji |
|-----------|--------|-------|
| actual ≥ expected × 1.05 | AHEAD | ✅ |
| expected × 0.95 ≤ actual < expected × 1.05 | ON TRACK | ➡️ |
| actual < expected × 0.95 | BEHIND | ⚠️ |

> ⚠️ **WARNING — Pacing Bug (Fixed in L3)**
> The pacing badge must compare against the expected value **for today's date**, not against the final target. If you compare actual ($1M) vs. total target ($2.5M) = "Behind" always until the last day. The correct comparison is actual ($1M) vs. expected-for-today ($1.25M on Day 10 of 20).

### Pacing Display Format

```
📏 PACING: BEHIND ⚠️
• Expected by Day 10: $1,250,000 / 190 sales
• Actual: $1,049,000 / 171 sales
• Gap: -$201,000 / -19 sales
• To hit target: need $145K/day for remaining 10 days (vs $125K/day pace)
```

**Include "catch-up pace" when behind:** Calculate the daily rate needed for remaining days to still hit target.

```python
if actual_revenue < expected_revenue:
    remaining_days = total_selling_days - days_elapsed
    remaining_target = target_revenue - actual_revenue
    if remaining_days > 0:
        catchup_daily = remaining_target / remaining_days
        normal_daily = target_revenue / total_selling_days
        # "Need $145K/day for remaining 10 days (vs $125K/day normal pace)"
```

---

## What NOT to Include

The brief must be signal, not noise. Exclude:

| Exclude | Why |
|---------|-----|
| Routine cron completions | "Sync ran successfully" is noise unless it failed |
| Tasks already completed and communicated | Don't re-report |
| Detailed technical logs | Keep it executive-level |
| Speculative predictions | Unless backed by data |
| Things that don't need action | If no one needs to do anything, don't mention it |
| Repeated information | If it was in yesterday's brief and hasn't changed, skip |
| "No issues" filler lines | Only flag FLAGS if there are flags |
| Long explanations | Bullet points. One line each. |

**Jaydyn's rule: "Don't repeat back what he said."** If he gave an instruction yesterday and you're executing it, don't say "As per your instruction yesterday..." Just show the result.

---

## Voice Note Option (TTS)

Jaydyn prefers voice note summaries he can listen to on the go. Use ElevenLabs with the Jarvis MCU voice (cloned Paul Bettany).

### TTS Format Rules

1. **Keep under 1,500 characters** (roughly 90 seconds of speech)
2. **Strip all emojis** (they're read aloud: "chart increasing" for 📈)
3. **Strip markdown formatting** (no asterisks, no hashes)
4. **Use natural language** instead of tables
5. **Numbers should be spoken naturally** ("two million one fifty-one thousand" not "$2,151,000")
6. **Lead with the headline**, then key details, then one insight

### TTS Script Template

```
Good morning. Here's your Day [N] brief for [Launch Name].

Revenue is at [X] dollars, [Y] percent of the [target] target.
[X] sales so far with [X] PIFs.
Pacing is [ahead/behind/on track] — we expected [X] by today and we're at [Y].

Yesterday brought in [X] new sales worth [Y] dollars. Top closer was [Rep] with [X] sales.

[One programme insight — e.g., "AI conversion is still the weakest at 25 percent. Consider a dedicated closer."]

[One flag if any — e.g., "Heads up: the PIF sync had an error last night, investigating."]

That's it. Have a good one.
```

### Generating the Voice Note

```python
# Using ElevenLabs API
import requests

def generate_voice_brief(text):
    response = requests.post(
        "https://api.elevenlabs.io/v1/text-to-speech/<VOICE_ID>",
        headers={
            "xi-api-key": "<ELEVENLABS_API_KEY>",
            "Content-Type": "application/json"
        },
        json={
            "text": text,
            "model_id": "eleven_multilingual_v2",
            "voice_settings": {
                "stability": 0.5,
                "similarity_boost": 0.75
            }
        }
    )
    # Save as MP3, send via WhatsApp
    with open("/tmp/daily_brief.mp3", "wb") as f:
        f.write(response.content)
```

Or use the OpenClaw `tts` tool:
```
tts(text="Good morning. Here's your Day 10 brief...", channel="telegram")
```

---

## Delivery Workflow

### Automated (Cron) Flow

1. **5:30 AM AEST** — Run GHL/Supabase sync (if launch is active)
2. **5:45 AM AEST** — Pull all data sources (sales, Slack, cron health, tasks)
3. **5:55 AM AEST** — Compile brief using template
4. **6:00 AM AEST** — Send WhatsApp text to Jaydyn
5. **6:01 AM AEST** — (Optional) Send voice note to Jaydyn
6. **6:05 AM AEST** — Post to Slack #ops or #daily-brief

### Manual Flow

When generating a brief on demand:
1. Pull latest data from Supabase/GHL
2. Check Slack for recent activity
3. Check cron health
4. Compile and format
5. Deliver via requested channel

---

## Handling Edge Cases

### No New Sales Yesterday
```
📅 YESTERDAY
• No new sales closed. Pipeline activity: [X] new opps, [Y] shows scheduled today.
```
Don't pretend nothing happened — show pipeline movement instead.

### Weekend Brief
Keep it short. Only include if:
- An active launch is running
- A system issue occurred
- Something time-sensitive needs Monday action

### Data Sync Failure
If sales data is stale:
```
⚠️ DATA NOTE: GHL sync last successful at [TIME]. Numbers may be [X] hours stale. Investigating.
```
**Never present stale data as current without flagging it.**

### Timezone Considerations
- Jaydyn is Brisbane GMT+10
- Calvin is Perth GMT+8
- Sales reps are across multiple timezones
- "Yesterday" means midnight-to-midnight AEST for metric calculation
- GHL timestamps are typically UTC — convert before comparing

---

## Quality Checklist

Before delivering any brief:

```
[ ] All numbers are current (synced within last 6 hours)
[ ] Pacing calculation uses today's expected, not total target
[ ] Programme breakdown adds up to total (sanity check)
[ ] Rep names are spelled correctly (use normalised roster)
[ ] No stale data presented without a flag
[ ] Under 4,000 chars for WhatsApp
[ ] No markdown tables for WhatsApp (use bullet lists)
[ ] Text sent as separate message from any attachments
[ ] TTS version (if used) is under 1,500 chars with no emojis
[ ] Only actionable flags included — no noise
```

---

## Example Brief (L3, Day 10)

```
📊 L3 Day 10/20 — Tuesday Apr 1

💰 Revenue: $1,049,000 AUD (42% of $2.5M)
📈 Sales: 171 (45% of 379)
💎 PIFs: 41 (54% of 76)
📋 Opps: 987

📏 PACING: BEHIND ⚠️
• Expected by Day 10: $1,250,000 / 190 sales
• Actual: $1,049,000 / 171 sales
• Gap: -$201K / -19 sales
• Need $145K/day for remaining 10 days (vs $125K normal pace)

📅 YESTERDAY (+15 sales, +$98K)
• Mandy Foxton: 3 sales ($21,800) — MOC, HD
• Daniel Lewis: 3 sales ($23,400) — Biz
• Gil Nelson: 2 sales ($14,600) — HD, Biz
• 7 other reps: 7 sales combined

📊 PROGRAMME STANDING
• Business: 48 sales / $379K (strongest)
• MOC: 39 sales / $275K
• HD: 32 sales / $215K
• AI: 21 sales / $143K (weakest conversion)
• Astrology: 18 sales / $85K
• Wellness: 13 sales / $60K

🚩 FLAGS
• AI conversion at 24% vs 35%+ for other programmes. 114 not-yet-called in AI pipeline.

💡 INSIGHT
• We crossed $1M at the 6am sync. Day 10 of 20 = exactly halfway. Revenue needs to accelerate — the Week 1 pace ($140K/day) vs Week 2 ($65K/day) suggests a mid-campaign activation push would help.
```

---

*This skill is self-contained. A fresh agent can generate and deliver a complete daily operations brief using only this document.*
