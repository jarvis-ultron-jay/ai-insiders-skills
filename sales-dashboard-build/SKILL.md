---
name: sales-dashboard-build
description: Build a real-time sales dashboard — Next.js frontend on Vercel, Supabase data layer, GHL API sync, Stripe verification. Includes pacing charts, rep leaderboard, programme breakdown, daily progression, and cron automation.
---
# Sales Dashboard Build

**Version:** 1.0.0
**Description:** Build and maintain a real-time sales dashboard for Wild Success launch campaigns. Covers the complete stack: Vercel (Next.js) frontend, Supabase (PostgreSQL) data layer, GHL API for pipeline data, Stripe API for payment verification. Includes data model schemas, sync architecture, dashboard components, pacing logic, rep leaderboard, daily progression charts, mobile responsive design, and cron automation.

**Trigger phrases:** "sales dashboard", "build dashboard", "launch dashboard", "real-time dashboard", "sales tracker", "dashboard components", "pacing chart", "rep leaderboard", "daily progression", "dashboard sync", "mission control"

---

## Context: Why a Sales Dashboard?

Wild Success runs 3-4 launch campaigns per year, each generating $1.5M-$3M+ AUD in revenue over 2-4 weeks. During a live launch, the exec team and sales team need real-time visibility into:
- Total sales and revenue (vs. target)
- Pacing — ahead or behind the linear interpolation line
- Per-programme breakdown
- Per-rep leaderboard
- Daily progression trend
- PIF (Paid in Full) vs. Payment Plan split

GHL's native reporting is inadequate for this. Hence, a custom dashboard pulling from GHL into Supabase and displaying via a Next.js app on Vercel.

---

## Section 1: Architecture

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   GHL API   │────▶│   Sync Job   │────▶│   Supabase   │────▶│   Vercel     │
│  (9 sub-    │     │  (Cron/Node) │     │  (PostgreSQL)│     │  (Next.js)   │
│   accounts) │     │              │     │              │     │              │
└─────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                               ▲
                                               │
                    ┌──────────────┐            │
                    │  Stripe API  │────────────┘
                    │  (Payment    │  (Optional: payment
                    │   verify)    │   verification sync)
                    └──────────────┘
```

### Component Roles

| Component | Tech | Purpose |
|-----------|------|---------|
| **Data Source** | GHL v2 API | Pipeline opportunities, contacts, tags (sales data) |
| **Data Source** | Stripe API | Payment verification, revenue confirmation (optional) |
| **Sync Layer** | Node.js / Python script via cron | Pull from APIs, transform, upsert to Supabase |
| **Data Layer** | Supabase (PostgreSQL) | Persistent storage, queryable, API via PostgREST |
| **Frontend** | Next.js (App Router) on Vercel | Dashboard UI, charts, real-time display |
| **Hosting** | Vercel | Edge deployment, auto-SSL, preview deploys |

### Alternative Architecture (Convex)

Wild Success also uses Convex for some real-time applications (Mission Control). If using Convex instead of Supabase for the frontend data layer:

```
GHL → Sync Script → Convex (real-time DB) → Next.js (Vercel)
```

Convex provides real-time subscriptions (data updates push to the frontend without polling). Use Convex when real-time updates matter (e.g., during live launch events).

---

## Section 2: Data Model

### 2.1 Supabase Tables

#### opportunities

```sql
CREATE TABLE opportunities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  opportunity_id TEXT UNIQUE NOT NULL,
  contact_id TEXT,
  contact_name TEXT,
  contact_email TEXT,
  pipeline_id TEXT NOT NULL,
  pipeline_name TEXT,
  stage_id TEXT,
  stage_name TEXT,
  stage_category TEXT CHECK (stage_category IN ('won', 'open', 'lost')),
  programme TEXT NOT NULL,
  launch TEXT NOT NULL,
  monetary_value NUMERIC DEFAULT 0,
  currency TEXT DEFAULT 'AUD',
  assigned_to TEXT,
  is_pif BOOLEAN DEFAULT FALSE,
  pif_source TEXT,
  source TEXT,
  status TEXT,
  location_id TEXT,
  created_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ,
  synced_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_opportunities_launch ON opportunities(launch);
CREATE INDEX idx_opportunities_programme ON opportunities(programme);
CREATE INDEX idx_opportunities_stage_category ON opportunities(stage_category);
CREATE INDEX idx_opportunities_assigned_to ON opportunities(assigned_to);
```

#### daily_progression

```sql
CREATE TABLE daily_progression (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  launch TEXT NOT NULL,
  date DATE NOT NULL,
  day_number INTEGER NOT NULL,
  total_sales INTEGER DEFAULT 0,
  total_revenue NUMERIC DEFAULT 0,
  total_pifs INTEGER DEFAULT 0,
  total_opps INTEGER DEFAULT 0,
  programme_breakdown JSONB DEFAULT '{}',
  rep_breakdown JSONB DEFAULT '{}',
  pacing_expected_revenue NUMERIC,
  pacing_expected_sales INTEGER,
  pacing_status TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(launch, date)
);

CREATE INDEX idx_daily_progression_launch ON daily_progression(launch);
```

#### launch_archives

```sql
CREATE TABLE launch_archives (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  launch TEXT UNIQUE NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  selling_days INTEGER NOT NULL,
  target_revenue NUMERIC NOT NULL,
  target_sales INTEGER NOT NULL,
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
  archived_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### launch_config (active campaign configuration)

```sql
CREATE TABLE launch_config (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  launch TEXT UNIQUE NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  selling_days INTEGER NOT NULL,
  target_revenue NUMERIC NOT NULL,
  target_sales INTEGER NOT NULL,
  target_pifs INTEGER,
  programmes JSONB NOT NULL,  -- [{name, pipeline_id, location_id, target_sales, target_revenue}]
  rep_roster JSONB,           -- [{name, normalised_name}]
  stage_maps JSONB NOT NULL,  -- {pipeline_id: {stage_id: {name, category}}}
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 2.2 Stripe Tables (Optional)

```sql
CREATE TABLE stripe_payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stripe_payment_id TEXT UNIQUE NOT NULL,
  customer_email TEXT,
  amount NUMERIC,
  currency TEXT DEFAULT 'aud',
  status TEXT,
  product_name TEXT,
  created_at TIMESTAMPTZ,
  synced_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## Section 3: Sync Layer

### 3.1 Sync Frequency

| Campaign State | Sync Frequency | Why |
|---------------|----------------|-----|
| Active launch (selling days) | Every 60 minutes (6am-11pm AEST) | Real-time visibility during campaign |
| Active launch (peak days) | Every 30 minutes | Last 3 days of campaign |
| Between launches | Once daily (6am AEST) | Baseline monitoring |
| Campaign closed | Disabled | Prevent stale data confusion |

### 3.2 Sync Script Structure

```javascript
// sync-pipeline.js — Node.js sync script
const { createClient } = require('@supabase/supabase-js');

const SUPABASE_URL = process.env.SUPABASE_URL;
const SUPABASE_KEY = process.env.SUPABASE_SERVICE_KEY;
const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);

async function syncPipeline(config) {
  const { locationId, pitToken, pipelineId, programme, launch, stageMap } = config;
  
  let allOpps = [];
  let startAfterId = null;
  let startAfter = null;
  let page = 0;
  
  // Paginate through all opportunities
  while (true) {
    page++;
    const payload = {
      location_id: locationId,
      pipeline_id: pipelineId,
      limit: 100
    };
    
    if (startAfterId && startAfter) {
      payload.startAfterId = startAfterId;
      payload.startAfter = startAfter;
    }
    
    const response = await fetch(
      'https://services.leadconnectorhq.com/opportunities/search',
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${pitToken}`,
          'Version': '2021-07-28',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(payload)
      }
    );
    
    if (response.status === 429) {
      // Rate limited — exponential backoff
      const wait = Math.min(60000, Math.pow(2, page) * 1000 + Math.random() * 1000);
      console.log(`Rate limited, waiting ${wait}ms`);
      await new Promise(r => setTimeout(r, wait));
      page--; // Retry same page
      continue;
    }
    
    const data = await response.json();
    const opps = data.opportunities || [];
    
    if (opps.length === 0) break;
    
    // Verify pagination advancement
    if (allOpps.length > 0 && opps[0].id === allOpps[allOpps.length - 100]?.id) {
      throw new Error('Pagination stuck — same first ID returned');
    }
    
    allOpps.push(...opps);
    
    const lastOpp = opps[opps.length - 1];
    startAfterId = lastOpp.id;
    startAfter = lastOpp.updatedAt || lastOpp.lastUpdatedAt;
    
    // Rate limit delay
    await new Promise(r => setTimeout(r, 700));
    
    if (opps.length < 100) break;
  }
  
  console.log(`${programme}: Fetched ${allOpps.length} opportunities`);
  
  // Process and upsert
  const processed = allOpps.map(opp => {
    const stageId = opp.pipelineStageId;
    const stageInfo = stageMap[stageId] || { name: 'UNKNOWN', category: 'unknown' };
    
    if (stageInfo.category === 'unknown') {
      console.warn(`⚠️ Unknown stage ${stageId} — opportunity ${opp.id} will be dropped!`);
    }
    
    return {
      opportunity_id: opp.id,
      contact_id: opp.contact?.id,
      contact_name: opp.contact?.name || opp.name,
      contact_email: opp.contact?.email,
      pipeline_id: pipelineId,
      pipeline_name: programme,
      stage_id: stageId,
      stage_name: stageInfo.name,
      stage_category: stageInfo.category,
      programme: programme,
      launch: launch,
      monetary_value: extractRevenue(opp.monetaryValue),
      currency: 'AUD',
      assigned_to: normaliseRepName(opp.assignedTo),
      is_pif: false,  // Will be updated by PIF detection pass
      location_id: locationId,
      created_at: opp.createdAt,
      updated_at: opp.updatedAt,
      synced_at: new Date().toISOString()
    };
  }).filter(opp => opp.stage_category !== 'unknown');
  
  // Batch upsert to Supabase
  const batchSize = 100;
  for (let i = 0; i < processed.length; i += batchSize) {
    const batch = processed.slice(i, i + batchSize);
    const { error } = await supabase
      .from('opportunities')
      .upsert(batch, { onConflict: 'opportunity_id' });
    
    if (error) {
      console.error(`Upsert error (batch ${Math.floor(i/batchSize) + 1}):`, error);
      throw error;
    }
  }
  
  return processed;
}

function extractRevenue(monetaryValue) {
  if (!monetaryValue) return 0;
  const val = parseFloat(monetaryValue);
  return val > 50000 ? val / 100 : val;  // Handle cents vs dollars
}

function normaliseRepName(name) {
  if (!name) return 'Unassigned';
  // Capitalise each word, trim whitespace
  return name.trim().split(' ').map(w => 
    w.charAt(0).toUpperCase() + w.slice(1).toLowerCase()
  ).join(' ');
}
```

### 3.3 PIF Detection Pass

After syncing opportunities, run a separate PIF detection pass for won opportunities:

```javascript
async function detectPIFs(launch, locationConfigs) {
  // Get all won opportunities that haven't been PIF-checked
  const { data: wonOpps } = await supabase
    .from('opportunities')
    .select('*')
    .eq('launch', launch)
    .eq('stage_category', 'won')
    .eq('pif_source', null);
  
  for (const opp of wonOpps) {
    const config = locationConfigs[opp.location_id];
    if (!config) continue;
    
    // Fetch contact tags from GHL
    const contactResp = await fetch(
      `https://services.leadconnectorhq.com/contacts/${opp.contact_id}`,
      {
        headers: {
          'Authorization': `Bearer ${config.pitToken}`,
          'Version': '2021-07-28'
        }
      }
    );
    
    await new Promise(r => setTimeout(r, 700)); // Rate limit
    
    const contactData = await contactResp.json();
    const tags = contactData.contact?.tags || [];
    
    const isPif = tags.some(t => t.toLowerCase().includes('pif'));
    
    await supabase
      .from('opportunities')
      .update({ is_pif: isPif, pif_source: 'tag' })
      .eq('opportunity_id', opp.opportunity_id);
  }
}
```

> ⚠️ **WARNING — NEVER use dollar thresholds for PIF detection.** Always use contact tags. See `ghl-pipeline-sync` skill for details.

### 3.4 Daily Progression Snapshot

Run once at end of each selling day (11 PM AEST):

```javascript
async function saveDailyProgression(launch, launchConfig) {
  const today = new Date().toISOString().split('T')[0]; // YYYY-MM-DD
  const dayNumber = calculateDayNumber(today, launchConfig.start_date);
  
  // Aggregate current data
  const { data: opps } = await supabase
    .from('opportunities')
    .select('*')
    .eq('launch', launch);
  
  const wonOpps = opps.filter(o => o.stage_category === 'won');
  
  const snapshot = {
    launch,
    date: today,
    day_number: dayNumber,
    total_sales: wonOpps.length,
    total_revenue: wonOpps.reduce((sum, o) => sum + (o.monetary_value || 0), 0),
    total_pifs: wonOpps.filter(o => o.is_pif).length,
    total_opps: opps.length,
    programme_breakdown: aggregateByField(wonOpps, 'programme'),
    rep_breakdown: aggregateByField(wonOpps, 'assigned_to'),
    pacing_expected_revenue: launchConfig.target_revenue * (dayNumber / launchConfig.selling_days),
    pacing_expected_sales: Math.round(launchConfig.target_sales * (dayNumber / launchConfig.selling_days)),
    pacing_status: calculatePacingStatus(/* ... */)
  };
  
  await supabase
    .from('daily_progression')
    .upsert(snapshot, { onConflict: 'launch,date' });
}
```

---

## Section 4: Dashboard Components

### 4.1 Campaign Overview Card

**Displays:** Total sales, revenue, PIFs, opps vs. targets. Pacing badge (Ahead/On Track/Behind).

```
┌─────────────────────────────────────────────────────┐
│  L3 — Day 15/20                                      │
│                                                       │
│  💰 $1,647,000 AUD     📈 250 Sales     💎 67 PIFs  │
│     65.9% of $2.5M        65.9% of 379     88% of 76│
│                                                       │
│  📏 Pacing: BEHIND ⚠️                                │
│  Expected: $1,875,000 / 284 sales                    │
│  Gap: -$228,000 / -34 sales                          │
└─────────────────────────────────────────────────────┘
```

### 4.2 Pacing Chart (Line Chart)

```
Revenue ($)
  │
2.5M ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ TARGET
  │                                           /
  │                                      /  /
  │                                 /  /
  │                            /  /    ← Pacing line (linear)
  │                       /  /
  │                  /  / ← Actual revenue
  │             /  /
  │        / /
  │   / /
  │ /
  └─────────────────────────────────────────────── Day
   1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20
```

**Implementation:** Use Chart.js, Recharts, or similar library. Two lines:
1. **Pacing line:** Linear from $0 on Day 0 to target on Day 20 (dashed)
2. **Actual line:** Plot from `daily_progression` table (solid)

> ⚠️ **WARNING — Pacing Logic Bug (Fixed in L3)**
> The "Behind/Ahead" badge must compare actual vs. expected **for today's date on the pacing line**, not actual vs. final target.
>
> **Wrong:** `actualRevenue < targetRevenue` → always "Behind" until final day
> **Correct:** `actualRevenue < expectedRevenueForToday` where `expectedRevenueForToday = target × (dayNumber / totalDays)`

### 4.3 Programme Breakdown (Bar Chart or Table)

| Programme | Sales | Revenue AUD | PIFs | PIF% | Conv Rate | % of Total |
|-----------|-------|-------------|------|------|-----------|------------|
| Business | 93 | $735,900 | 10 | 10.8% | 53.1% | 29.5% |
| MOC | 78 | $550,200 | 17 | 21.8% | 48.8% | 24.8% |
| HD | 58 | $389,700 | 23 | 39.7% | 39.5% | 18.4% |
| AI | 34 | $232,200 | 9 | 26.5% | 25.4% | 10.8% |
| Astrology | 36 | $169,200 | 25 | 69.4% | 35.3% | 11.4% |
| Wellness | 16 | $73,800 | 10 | 62.5% | 25.0% | 5.1% |

Conversion rate = won / (won + showed + not yet called)

### 4.4 Rep Leaderboard

Sortable table showing per-rep performance:

| Rank | Rep | Sales | Revenue AUD | PIFs | Programmes | Conv% |
|------|-----|-------|-------------|------|------------|-------|
| 1 | Mandy Foxton | 37 | $268,800 | 5 | MOC, HD, AI, Biz, Well | — |
| 2 | Anne Breakwell | 32 | $224,400 | 9 | MOC, HD, Biz | — |
| 3 | Gil Nelson | 32 | $217,200 | 7 | MOC, HD, AI, Biz, Astro | — |

**Data query:**

```sql
SELECT 
  assigned_to AS rep,
  COUNT(*) AS sales,
  SUM(monetary_value) AS revenue,
  COUNT(*) FILTER (WHERE is_pif) AS pifs,
  ARRAY_AGG(DISTINCT programme) AS programmes
FROM opportunities
WHERE launch = 'L3' AND stage_category = 'won'
GROUP BY assigned_to
ORDER BY revenue DESC;
```

### 4.5 Daily Progression (Area Chart)

Show cumulative sales and revenue day-over-day:

```sql
SELECT date, day_number, total_sales, total_revenue, total_pifs
FROM daily_progression
WHERE launch = 'L3'
ORDER BY date ASC;
```

Each day is a data point. Area under the curve filled. Pacing line overlaid.

### 4.6 PIF Tracker

Dedicated view for PIF (Paid in Full) tracking:
- Total PIFs vs. target
- PIF percentage (PIFs / total sales)
- PIF by programme
- PIF by rep
- PIF cash collected vs. PP initial payments

### 4.7 Not-Yet-Called (NYC) Pipeline

Show opportunities in "Not Yet Called" stages:

```sql
SELECT programme, COUNT(*) AS nyc_count
FROM opportunities
WHERE launch = 'L3' AND stage_name ILIKE '%not yet called%'
GROUP BY programme
ORDER BY nyc_count DESC;
```

This identifies untapped pipeline. During L3, 350 opps (25.5%) were never called.

---

## Section 5: Frontend Implementation

### 5.1 Next.js App Structure

```
src/
  app/
    page.tsx                 # Main dashboard entry
    layout.tsx               # Root layout with nav
    api/
      sync/route.ts          # API route for manual sync trigger
  components/
    CampaignOverview.tsx     # Headline numbers + pacing badge
    PacingChart.tsx           # Revenue vs pacing line chart
    ProgrammeBreakdown.tsx   # Per-programme table/chart
    RepLeaderboard.tsx       # Sortable rep table
    DailyProgression.tsx     # Day-by-day area chart
    PifTracker.tsx           # PIF-specific metrics
    NycPipeline.tsx          # Not-yet-called counts
  lib/
    supabase.ts              # Supabase client config
    queries.ts               # Data fetching functions
    pacing.ts                # Pacing calculation utilities
    types.ts                 # TypeScript interfaces
```

### 5.2 Supabase Client Setup

```typescript
// src/lib/supabase.ts
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```

### 5.3 Pacing Utility

```typescript
// src/lib/pacing.ts
export interface PacingResult {
  expectedRevenue: number;
  expectedSales: number;
  actualRevenue: number;
  actualSales: number;
  revenueGap: number;
  salesGap: number;
  status: 'ahead' | 'on-track' | 'behind';
  catchUpDailyRate?: number;
}

export function calculatePacing(
  targetRevenue: number,
  targetSales: number,
  totalSellingDays: number,
  daysElapsed: number,
  actualRevenue: number,
  actualSales: number
): PacingResult {
  // Cap days elapsed at total selling days
  const capped = Math.min(daysElapsed, totalSellingDays);
  
  const expectedRevenue = targetRevenue * (capped / totalSellingDays);
  const expectedSales = Math.round(targetSales * (capped / totalSellingDays));
  
  const revenueGap = actualRevenue - expectedRevenue;
  const salesGap = actualSales - expectedSales;
  
  let status: 'ahead' | 'on-track' | 'behind';
  if (actualRevenue >= expectedRevenue * 1.05) {
    status = 'ahead';
  } else if (actualRevenue >= expectedRevenue * 0.95) {
    status = 'on-track';
  } else {
    status = 'behind';
  }
  
  // Calculate catch-up rate if behind
  let catchUpDailyRate: number | undefined;
  if (status === 'behind') {
    const remainingDays = totalSellingDays - capped;
    if (remainingDays > 0) {
      catchUpDailyRate = (targetRevenue - actualRevenue) / remainingDays;
    }
  }
  
  return {
    expectedRevenue,
    expectedSales,
    actualRevenue,
    actualSales,
    revenueGap,
    salesGap,
    status,
    catchUpDailyRate
  };
}
```

### 5.4 Mobile Responsive Design

Dashboard must be usable on mobile (Jaydyn checks on his phone).

**Requirements:**
- Cards stack vertically on screens < 768px
- Tables scroll horizontally on small screens
- Charts resize proportionally (use responsive chart options)
- Font size minimum 14px for readability
- Touch-friendly tap targets (minimum 44x44px)
- No hover-only interactions

```css
/* Key responsive breakpoints */
@media (max-width: 768px) {
  .dashboard-grid {
    grid-template-columns: 1fr;  /* Stack all cards */
  }
  .data-table {
    overflow-x: auto;  /* Horizontal scroll */
    -webkit-overflow-scrolling: touch;
  }
}
```

---

## Section 6: Deployment

### 6.1 Vercel Deployment

```bash
cd /Users/jarvis/Projects/wild-success-hq  # or wherever the dashboard lives

# Deploy to production
npx vercel --yes --prod

# Environment variables (set in Vercel dashboard):
# NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
# NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
# SUPABASE_SERVICE_KEY=eyJ...  (server-side only, for API routes)
```

### 6.2 If Using Convex (Alternative)

```bash
# Deploy Convex schema + functions FIRST
npx convex deploy --cmd 'npm run build' --yes

# Then deploy Vercel frontend
npx vercel --yes --prod
```

**Gotcha:** Vercel filesystem is read-only. Don't write to /tmp on cold starts. Use Supabase or Convex for all persistence.

### 6.3 Vercel Environment Variables

Set via Vercel dashboard or CLI:

```bash
vercel env add NEXT_PUBLIC_SUPABASE_URL production
vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY production
vercel env add SUPABASE_SERVICE_KEY production
```

---

## Section 7: Cron Setup

### 7.1 OpenClaw Cron Jobs

```bash
# Hourly pipeline sync during active launch (6am-11pm AEST)
openclaw cron add --name "Launch Pipeline Sync" \
  --schedule "0 6-23 * * *" \
  --timezone "Australia/Brisbane" \
  --command "node /path/to/sync-pipeline.js"

# Daily progression snapshot (11pm AEST)
openclaw cron add --name "Daily Progression Snapshot" \
  --schedule "0 23 * * *" \
  --timezone "Australia/Brisbane" \
  --command "node /path/to/daily-snapshot.js"

# PIF detection (every 2 hours during launch)
openclaw cron add --name "PIF Detection" \
  --schedule "0 */2 * * *" \
  --timezone "Australia/Brisbane" \
  --command "node /path/to/detect-pifs.js"
```

### 7.2 Campaign Lifecycle

```
Campaign Start:
  → Enable: Pipeline Sync, Daily Snapshot, PIF Detection crons
  → Configure: launch_config table with targets and stage maps
  → Verify: Run first sync manually, check data in dashboard

Campaign Active:
  → Monitor: Sync health, data freshness, dashboard accuracy
  → Adjust: Increase sync frequency in final 3 days

Campaign End:
  → Run: Final sync
  → Disable: ALL launch-specific crons immediately
  → Archive: Insert into launch_archives table
  → Freeze: Daily progression data
```

> ⚠️ **WARNING — Disable crons IMMEDIATELY after campaign close.** Stale crons posting old data to Slack or updating dashboards with frozen numbers causes confusion and erodes trust.

---

## Section 8: Data Verification

### 8.1 Post-Sync Checks

After every sync, run these validations:

```sql
-- 1. Total won count per programme (should match GHL)
SELECT programme, COUNT(*) AS won_count, SUM(monetary_value) AS revenue
FROM opportunities
WHERE launch = 'L3' AND stage_category = 'won'
GROUP BY programme;

-- 2. No programme showing 0 that had data yesterday
WITH today AS (
  SELECT programme, COUNT(*) as cnt
  FROM opportunities WHERE launch = 'L3' AND stage_category = 'won'
  GROUP BY programme
),
yesterday AS (
  SELECT (programme_breakdown)::jsonb AS breakdown
  FROM daily_progression
  WHERE launch = 'L3' AND date = CURRENT_DATE - 1
)
-- Compare and flag any drops to zero

-- 3. PIF counts are reasonable
SELECT programme, COUNT(*) FILTER (WHERE is_pif) AS pifs,
       COUNT(*) AS total_won,
       ROUND(COUNT(*) FILTER (WHERE is_pif)::numeric / NULLIF(COUNT(*), 0) * 100, 1) AS pif_pct
FROM opportunities
WHERE launch = 'L3' AND stage_category = 'won'
GROUP BY programme;

-- 4. Daily sales should be monotonically increasing (unless RTCs)
SELECT date, total_sales,
       total_sales - LAG(total_sales) OVER (ORDER BY date) AS daily_delta
FROM daily_progression
WHERE launch = 'L3'
ORDER BY date;
```

### 8.2 Alert on Failures

If any check fails, immediately alert:
- WhatsApp to Jaydyn (for data trust issues)
- Slack #ops (for team visibility)

```
⚠️ DATA ALERT: [Programme] showing 0 sales but had [X] yesterday. Investigating sync.
```

> **"If I was to believe that, or I just sent this information to Calvin, I'm the one who looks like a dickhead."** — Jaydyn
>
> Data accuracy is a trust issue. Wrong data on the dashboard = reputational risk for Jaydyn with Calvin. This is non-negotiable.

---

## Section 9: Stripe Integration (Optional)

### 9.1 Purpose

Cross-reference GHL sales with Stripe payments for revenue verification.

### 9.2 Stripe API

```bash
# List charges for a date range
curl "https://api.stripe.com/v1/charges?created[gte]=1711234800&created[lte]=1712444400&limit=100" \
  -H "Authorization: Bearer <STRIPE_RESTRICTED_KEY>" \
  -H "Stripe-Version: 2024-12-18.acacia"
```

### 9.3 Access Level

Stripe is **READ-ONLY** (restricted key). Wild Success has 12 Stripe accounts under one org. Currency is AUD.

---

## Known Regressions & Gotchas

| ID | Description | Impact | Fix |
|----|-------------|--------|-----|
| T-11 | Missing stage in local stage map | Sales silently dropped | Verify ALL stages mapped before/during campaign |
| PIF-01 | Dollar threshold PIF detection | Wrong PIF counts | Use contact tags only |
| PAG-01 | GHL single-param pagination | Stuck on page 1 forever | Send BOTH startAfterId AND startAfter |
| PACE-01 | Pacing compared to final target | Always shows "Behind" | Compare to today's expected value |
| DB-01 | Missing conn.commit() | Data silently lost | Explicit commit after every batch |
| DASH-01 | Stale crons after campaign close | Confusion, wrong data | Disable ALL crons immediately after final sync |
| DASH-02 | findBestSyncDate() needed for PIF tracker | Partial sync shows incomplete data | Use findBestSyncDate() utility for date-dependent components |

---

*This skill is self-contained. A fresh agent can build and operate a complete sales dashboard using only this document.*
