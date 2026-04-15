---
name: cdp-design
description: Design and build a Customer Data Platform (CDP) using Supabase or similar. Guide users through architecture design, schema creation, data source integration, sync patterns, deduplication, reporting, security, and migration. Use when asked to build a CDP, unify customer data across systems, create a single source of truth for contacts, design a master contacts table, integrate CRM/payment/community data, or plan a data warehouse for customer metrics. Triggers on "CDP", "customer data platform", "single source of truth", "unify contacts", "master contacts", "data unification", "customer 360", "contact deduplication".
---

# CDP Design — Customer Data Platform Architecture

Methodology developed by WILD Success (112K+ contacts, 9 sub-accounts, multiple payment processors, community platforms, and CRM systems). Adapted for AI-assisted CDP design by AI Insiders.

## Core Philosophy

The CDP answers one question: **"Who is this person across all our systems?"**

Rules:
- Every external system syncs TO the CDP. The CDP never syncs outward as source.
- The CDP is the single source of truth for customer identity and metrics.
- No system owns the customer — the CDP does. CRMs, payment processors, and communities are data sources, not authorities.
- If two systems disagree about a customer, the CDP resolves it using defined precedence rules.

## Phase 1: Discovery

Before designing anything, map the landscape. Ask the user:

1. **What systems hold customer data today?** (CRM, payments, community, support, email, spreadsheets)
2. **How many contacts exist across all systems?** (rough count — 1K, 10K, 100K+)
3. **What's the primary identifier?** (email is most common; phone for SMS-heavy businesses)
4. **What questions can't you answer today?** (These become the reporting requirements)
5. **Who needs access?** (Sales, ops, finance, marketing — drives RLS design)
6. **What's the highest-value data source?** (Start migration here)

Output a **Source Map** — a table of every system, what data it holds, identifier used, estimated record count, and update frequency.

Example Source Map:

| System | Data Types | Identifier | Records | Update Frequency |
|--------|-----------|------------|---------|-----------------|
| GHL (CRM) | Contacts, pipelines, tags | Email + phone | 50K | Hourly |
| Stripe | Transactions, subscriptions | Email + customer_id | 12K | Real-time (webhooks) |
| Circle | Memberships, engagement | Email | 30K | Daily |
| Intercom | Tickets, conversations | Email | 8K | Daily |

## Phase 2: Schema Design

### Master Contacts Table

This is the heart of the CDP. Every person gets exactly one row.

```sql
CREATE TABLE contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  -- Identity
  email TEXT UNIQUE,
  email_secondary TEXT,
  phone TEXT,
  first_name TEXT,
  last_name TEXT,
  full_name TEXT GENERATED ALWAYS AS (first_name || ' ' || last_name) STORED,

  -- Source tracking
  primary_source TEXT NOT NULL,          -- which system created this record
  created_via TEXT,                       -- 'stripe_webhook', 'ghl_sync', 'manual', etc.

  -- Lifecycle
  status TEXT DEFAULT 'active',          -- active, churned, prospect, archived
  first_seen_at TIMESTAMPTZ DEFAULT NOW(),
  last_seen_at TIMESTAMPTZ DEFAULT NOW(),
  last_synced_at TIMESTAMPTZ,

  -- Computed / denormalized
  ltv_cents BIGINT DEFAULT 0,
  total_transactions INT DEFAULT 0,
  active_subscriptions INT DEFAULT 0,
  tags TEXT[] DEFAULT '{}',

  -- Metadata
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_contacts_email ON contacts(email);
CREATE INDEX idx_contacts_phone ON contacts(phone);
CREATE INDEX idx_contacts_status ON contacts(status);
```

### Source Identities Table

Maps external system IDs to the master contact. One contact can have many source identities.

```sql
CREATE TABLE source_identities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID REFERENCES contacts(id) ON DELETE CASCADE,
  source TEXT NOT NULL,           -- 'ghl', 'stripe', 'circle', 'intercom'
  external_id TEXT NOT NULL,      -- the ID in that system
  source_metadata JSONB,          -- extra fields from that system
  last_synced_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(source, external_id)
);
```

### Supporting Tables

Design these based on the user's reporting needs:

```sql
-- Transactions (from Stripe, PayPal, manual)
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID REFERENCES contacts(id),
  source TEXT NOT NULL,
  external_id TEXT UNIQUE,
  amount_cents BIGINT NOT NULL,
  currency TEXT DEFAULT 'AUD',
  type TEXT,                      -- 'payment', 'refund', 'dispute'
  product_name TEXT,
  status TEXT,                    -- 'succeeded', 'failed', 'refunded'
  occurred_at TIMESTAMPTZ NOT NULL,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Subscriptions
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID REFERENCES contacts(id),
  source TEXT NOT NULL,
  external_id TEXT UNIQUE,
  plan_name TEXT,
  amount_cents BIGINT,
  currency TEXT DEFAULT 'AUD',
  interval TEXT,                  -- 'month', 'year'
  status TEXT,                    -- 'active', 'canceled', 'past_due'
  started_at TIMESTAMPTZ,
  canceled_at TIMESTAMPTZ,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Interactions (community, support, engagement)
CREATE TABLE interactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID REFERENCES contacts(id),
  source TEXT NOT NULL,
  type TEXT NOT NULL,             -- 'support_ticket', 'community_post', 'email_open', 'login'
  occurred_at TIMESTAMPTZ NOT NULL,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Sync log (every sync run)
CREATE TABLE sync_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID NOT NULL,       -- groups all records from one sync run
  source TEXT NOT NULL,
  started_at TIMESTAMPTZ DEFAULT NOW(),
  finished_at TIMESTAMPTZ,
  records_processed INT DEFAULT 0,
  records_created INT DEFAULT 0,
  records_updated INT DEFAULT 0,
  records_errored INT DEFAULT 0,
  error_details JSONB,
  status TEXT DEFAULT 'running'   -- 'running', 'completed', 'failed'
);
```

Guide the user to add or remove tables based on their actual needs. Not every business needs all of these.

## Phase 3: Deduplication Strategy

Deduplication runs on every inbound sync. The matching order:

1. **Exact email match** → Highest confidence. Link to existing contact.
2. **Exact phone match** (normalized to E.164) → High confidence. Link if no email conflict.
3. **Fuzzy name + other signals** → Low confidence. Flag for manual review, do not auto-merge.

```
FUNCTION resolve_contact(email, phone, name):
  IF email AND exists(contacts WHERE email = input.email):
    RETURN existing contact
  IF phone AND exists(contacts WHERE phone = normalize(input.phone)):
    candidate = matching contact
    IF candidate.email IS NULL OR input.email IS NULL:
      RETURN candidate  -- no conflict, safe to merge
    ELSE:
      LOG warning "phone match but email mismatch" → manual review queue
  -- No match found: create new contact
  RETURN create_new_contact(email, phone, name)
```

Rules:
- Never auto-merge on name alone. Names are not unique.
- Log every merge decision with before/after state.
- Build a `merge_log` table for audit trail.
- Provide a manual review queue for low-confidence matches.

## Phase 4: Data Source Integration Patterns

### CRM Sync (GHL, HubSpot, Salesforce)

- **Method:** Polling (API pagination) on schedule
- **Frequency:** Every 1–4 hours
- **Data:** Contacts, pipeline stages, tags, custom fields
- **Key pattern:** Pull contacts modified since last sync (use `updated_after` or similar filter)
- **Watch for:** Rate limits (GHL: 100 req/min per location), pagination cursors, API version changes

### Payment Processor Sync (Stripe, PayPal)

- **Method:** Webhooks (preferred) + daily reconciliation poll
- **Frequency:** Real-time via webhooks; daily full reconciliation
- **Data:** Charges, refunds, subscriptions, invoices
- **Key pattern:** Webhook receives event → upsert transaction → update contact LTV
- **Watch for:** Webhook replay/retry (idempotency via event ID), out-of-order events, test mode leakage

### Community Platform Sync (Circle, Skool, Discord)

- **Method:** Polling (most lack webhooks)
- **Frequency:** Daily (engagement data is not time-critical)
- **Data:** Membership status, join date, activity level, posts/comments
- **Key pattern:** Pull member list, match to contacts by email, update engagement metrics

### Support Platform Sync (Intercom, Zendesk)

- **Method:** Webhooks (Intercom) or polling (Zendesk)
- **Frequency:** Daily for metrics; real-time webhooks for ticket events if available
- **Data:** Ticket count, resolution time, satisfaction score, conversation history
- **Key pattern:** Aggregate ticket metrics per contact, store summary not full transcripts

### Email/Marketing Sync

- **Method:** Polling or webhook depending on provider
- **Frequency:** Daily
- **Data:** Campaign sends, opens, clicks, unsubscribes, segment membership
- **Key pattern:** Store engagement scores, not individual event logs (unless needed for compliance)

## Phase 5: Sync Architecture

### Pull vs Push Decision Matrix

| Pattern | Use When | Pros | Cons |
|---------|---------|------|------|
| **Webhooks (push)** | Source supports them; data is time-sensitive (payments, tickets) | Real-time, lower API usage | Must handle retries, ordering, downtime |
| **Polling (pull)** | No webhook support; data is not time-sensitive; need full reconciliation | Simple, reliable, self-healing | Delayed, higher API usage, rate limit risk |
| **Hybrid** | Best of both — webhooks for real-time + daily poll for reconciliation | Comprehensive | More complex to build |

Recommend hybrid for any source that supports webhooks.

### Sync Session Pattern

Every sync run follows this structure:

```
1. Generate session_id (UUID)
2. Log sync_log entry: source, started_at, status='running'
3. Fetch data from source (paginated)
4. For each record:
   a. Resolve contact (dedup logic)
   b. Upsert source_identity
   c. Upsert domain record (transaction, subscription, interaction)
   d. Update contact denormalized fields (ltv, last_seen_at, etc.)
   e. Increment counters (processed, created, updated)
5. On error: log to error_details, increment records_errored, continue
6. Finalize: update sync_log with finished_at, final counts, status
```

### Idempotency

- Use `external_id` (the ID from the source system) as the unique key for upserts.
- `ON CONFLICT (source, external_id) DO UPDATE` — never creates duplicates.
- Webhook events: store `event_id` and skip if already processed.

### Conflict Resolution

When the same field is updated by multiple sources:

1. **Most recently updated wins** — compare timestamps.
2. **Source priority override** — some fields have an authoritative source (e.g., payment data from Stripe always wins over CRM payment fields).
3. **Audit trail** — store previous values in a `field_changes` JSONB log or separate audit table.

## Phase 6: Reporting Layer

### Key Metrics

Guide the user to build these based on their business model:

| Metric | Formula | Source Tables |
|--------|---------|--------------|
| **LTV** | SUM(transactions.amount_cents) WHERE status='succeeded' | transactions |
| **Churn Rate** | Contacts moving to status='churned' / total active, per period | contacts |
| **MRR** | SUM(subscriptions.amount_cents) WHERE status='active' AND interval='month' | subscriptions |
| **Acquisition Source** | GROUP BY contacts.primary_source | contacts |
| **Programme Completion** | Custom based on interaction milestones | interactions |
| **Payment Status** | Active subs, past due, canceled breakdown | subscriptions |

### Dashboard Patterns

**For real-time metrics** (MRR, active subs): Query live tables directly. Cache with Supabase materialized views if performance matters.

**For historical/aggregate metrics** (LTV trends, cohort analysis): Build aggregation tables updated on a schedule (daily cron).

```sql
-- Example: daily metrics snapshot
CREATE TABLE daily_metrics (
  date DATE PRIMARY KEY,
  total_contacts INT,
  active_contacts INT,
  mrr_cents BIGINT,
  new_contacts_today INT,
  churned_today INT,
  revenue_today_cents BIGINT,
  computed_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Cohort Analysis

Group contacts by shared attributes for trend analysis:

- **By acquisition date** — monthly cohorts: retention curves, LTV by cohort
- **By programme/product** — which product drives highest LTV?
- **By source** — which acquisition channel produces best customers?

Build cohort queries as Supabase database functions for reuse:

```sql
CREATE FUNCTION cohort_retention(cohort_month DATE, periods INT)
RETURNS TABLE(period INT, retained_count INT, retention_pct NUMERIC)
-- Implementation depends on user's specific lifecycle events
```

## Phase 7: Security & Access

### Row-Level Security (RLS)

Enable RLS on all tables. Design policies by role:

```sql
ALTER TABLE contacts ENABLE ROW LEVEL SECURITY;

-- Read access for all authenticated team members
CREATE POLICY "team_read" ON contacts FOR SELECT
  TO authenticated USING (true);

-- Write access restricted to sync service role
CREATE POLICY "sync_write" ON contacts FOR ALL
  TO service_role USING (true);
```

### Access Tiers

| Role | Read | Write | Tables |
|------|------|-------|--------|
| **Service role** (sync scripts) | All | All | All |
| **Ops/Admin** | All | Contacts, interactions | All |
| **Sales** | Contacts, transactions | None | Filtered by assignment |
| **Finance** | Transactions, subscriptions | None | All |
| **Marketing** | Contacts (no PII), interactions | None | Aggregates only |

### Credential Isolation

- **Service role key**: Used only by sync scripts running server-side. Never exposed to frontend.
- **Anon key + RLS**: Used by dashboards and team-facing apps. RLS enforces access.
- Store credentials in environment variables or a secrets manager. Never in code or workspace files.

### PII Handling

| Data | Store? | Hash? | Log? |
|------|--------|-------|------|
| Email | Yes (needed for dedup) | No | Never in plain text in logs |
| Phone | Yes (needed for dedup) | No | Never in plain text in logs |
| Full name | Yes | No | OK in audit logs |
| Payment details | No — let Stripe handle | N/A | Never |
| IP address | Only if compliance requires | Hash if storing | Never |
| Passwords | Never — not your system | N/A | Never |

## Phase 8: Migration Playbook

Execute in this exact order. Do not skip steps.

### Step 1: Build the Schema
Deploy all tables, indexes, RLS policies, and functions to Supabase. Test with manual inserts before any sync runs.

### Step 2: Pick Source #1
Choose the highest-value data source (usually CRM or payment processor). Build the sync script for this source only.

### Step 3: Initial Load
Run the first full sync. Validate:
- **Count check**: Records in CDP ≈ records in source (within 1% — some dedup expected)
- **Spot check**: Pick 10 random contacts, verify fields match source
- **Reconciliation**: Run a diff query to find missing or mismatched records

### Step 4: Add Sources One at a Time
For each additional source:
1. Build the sync script
2. Run initial load
3. Validate (count + spot + reconciliation)
4. Confirm dedup is working (check for new duplicates)
5. Only then proceed to next source

### Step 5: Enable Scheduled Sync
Once all sources are loaded and validated:
- Set up cron jobs (Supabase pg_cron, external scheduler, or edge functions)
- Monitor sync_log daily for the first two weeks
- Alert on: records_errored > 0, sync duration anomalies, count drift

### Step 6: Build Reporting
Only after data is flowing reliably:
- Build aggregation tables
- Create dashboard views
- Set up cohort analysis queries

**Critical rule: Never run all sources simultaneously on first sync.** Add one at a time. Validate each. This prevents cascading errors and makes debugging possible.

## Quick Reference: Recommended Sync Frequencies

| Source Type | Method | Frequency |
|-------------|--------|-----------|
| Payments (Stripe, PayPal) | Webhooks + daily reconciliation | Real-time + daily |
| CRM (GHL, HubSpot) | Polling (modified since) | Every 1–4 hours |
| Community (Circle, Skool) | Polling (full member list) | Daily |
| Support (Intercom, Zendesk) | Webhooks or polling | Daily |
| Email/Marketing | Polling | Daily |
| Manual/CSV imports | On-demand | As needed |

## Troubleshooting Guide

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Duplicate contacts appearing | Dedup not matching (email casing, phone format) | Normalize email to lowercase, phone to E.164 before matching |
| LTV doesn't match Stripe | Refunds not synced, or currency mismatch | Ensure refund webhooks are processed; verify currency conversion |
| Sync keeps timing out | Too many records per batch, rate limiting | Reduce batch size; add delays between API calls; use cursor pagination |
| Contact count keeps growing | New source creating contacts that should merge | Check dedup logic; verify identifier mapping for new source |
| sync_log shows errors but data looks fine | Non-critical errors (e.g., missing optional fields) | Review error_details; suppress non-critical warnings |

---

*Methodology developed by WILD Success. Adapted for AI-assisted CDP design by AI Insiders.*
