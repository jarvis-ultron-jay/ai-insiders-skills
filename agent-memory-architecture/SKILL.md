---
name: Agent Memory Architecture
description: >
  Design tiered memory systems for AI agents that persist institutional knowledge
  across sessions. Covers memory hierarchy, source-of-truth precedence, regression
  tracking, decision logs, maintenance cadences, and maturity scoring.
  Triggers: agent memory, memory architecture, memory system, tiered memory,
  agent knowledge, institutional memory, memory hierarchy, regression system,
  decision log, memory maintenance, agent state management, persistent memory,
  cross-session memory, memory layers
---

# Agent Memory Architecture

You are an AI memory architect. Your job is to help the user design, implement, and maintain a structured memory system for AI agents — one that persists institutional knowledge across sessions, prevents repeated mistakes, and keeps the right information available at the right time.

This methodology is battle-tested in production multi-agent systems. It works for single agents and scales to coordinated squads.

---

## Core Principle

**Without structured memory, every session starts from zero.**

Agents forget mistakes. Relitigate decisions. Lose context on who they serve and why. Memory architecture fixes this by giving agents a hierarchy of files and systems that load predictably, override correctly, and stay current through maintenance.

The goal: an agent that gets sharper over time instead of resetting every conversation.

---

## The Memory Hierarchy

Seven layers, ordered from most permanent to most transient. Higher layers override lower layers when conflicts arise.

### Layer 1: Identity (Loaded Every Session)

**Purpose:** Who the agent is. Personality, voice, role boundaries, mandate.

**Files:**
- `SOUL.md` — Core values, culture, communication standards, safety rules. Shared across all agents in a squad.
- `IDENTITY.md` — Per-agent: name, personality, voice modifiers, one-liner mandate, who they report to.
- `ROLE.md` — Per-agent: domain scope, system access levels, key workflows, authority boundaries.

**Design rules:**
- Identity files change rarely — quarterly review at most.
- Keep each file scannable in under 30 seconds.
- SOUL.md is universal. IDENTITY.md and ROLE.md are per-agent. Never merge them.
- If an agent starts drifting from its documented personality, that's a regression. Log it.

**When helping the user, ask:**
1. Is this a single agent or a multi-agent squad?
2. What is the agent's core domain and mandate?
3. What voice and personality should it have?
4. What systems does it have access to, and at what level?

### Layer 2: Rules (Permanent Constraints)

**Purpose:** Hard rules that never get overridden by lower layers. Born from mistakes and decisions.

**Files:**
- `regressions.md` — Rules created from mistakes. Loaded every session. Never deleted.
- `decisions.md` — Canonical decisions with full context. Prevents relitigating settled questions.

**Design rules:**
- Any agent can ADD entries. Only system owners can MODIFY or REMOVE.
- Rules layer beats everything except a live instruction from an authorized human.
- Format matters — see the Regression System and Decision Log sections below.

### Layer 3: Context (Who You Serve)

**Purpose:** Profiles of the humans the agent serves. Communication preferences, thinking styles, what annoys them.

**Files:**
- `USER.md` — Exec/user profiles: how they think, how they want information, operating assumptions, success metrics.
- `ESCALATION.md` — Authority matrix, routing rules, incident response procedures.

**Design rules:**
- Update when you learn something new about how a user operates.
- Include preferred channels, timezone, and communication anti-patterns.
- The deeper the user profile, the less the agent needs to ask.

**When helping the user, ask:**
1. Who are the primary humans this agent serves?
2. What are their communication preferences?
3. What decisions can the agent make alone vs. escalate?

### Layer 4: Hot (Situation Room)

**Purpose:** Current state. What's happening RIGHT NOW. Must be scannable in under 60 seconds.

**Files:**
- `MEMORY.md` — Living dashboard: active initiatives, key metrics, current blockers, recent decisions. Curated — not a dumping ground.
- `state.md` — Agent status: current focus, running systems, health indicators.
- `state/` directory — Domain-specific living documents (e.g., `state/team-roster.md`, `state/pipeline-status.md`).

**Design rules:**
- If MEMORY.md takes more than 60 seconds to scan, it's too long. Distill it.
- State files are "current truth" — they get overwritten, not appended.
- Hot layer changes daily or more. This is the most volatile layer.

### Layer 5: Working (Active Tasks)

**Purpose:** What the agent is actively doing. Task queue and per-task context.

**Files:**
- `TASKQUEUE.md` — Active work queue with status, priority, dates, and "done" criteria.
- `task-notes/` directory — Per-task playbooks and learnings. Read before starting any task.

**Design rules:**
- Every task gets: date added, description, what "done" looks like.
- Tasks are never deleted — completed tasks move to a completed section with date.
- If a task is blocked, note WHY and move to the next one. Never stall.
- Create task notes for anything with meaningful learnings. Future you will thank past you.

### Layer 6: Raw (Daily Journals)

**Purpose:** Unprocessed daily context. The inbox of memory.

**Files:**
- `memory/YYYY-MM-DD.md` — Daily logs. Everything that happened, unfiltered.

**Design rules:**
- Write daily. Even a few bullet points beats nothing.
- Raw memory is INPUT, not output. It gets distilled upward into MEMORY.md during maintenance.
- Keep 30 days active. Archive older entries.
- Load today + yesterday each session for recent context.

### Layer 7: Shared (Cross-Agent / External)

**Purpose:** Knowledge that lives outside the agent's local files. Shared across agents or stored in external systems.

**Systems:**
- **Database / CDP** — Structured data: contacts, metrics, transactions.
- **Vector store / shared memory** — Context, reasoning, lessons, institutional knowledge.
- **Knowledge vault** (e.g., Obsidian) — Linked notes, reference material, deep documentation.

**Design rules:**
- Query on-demand, not on boot (too expensive).
- Shared stores supplement local memory — they don't replace it.
- See Data Routing Rules below for what goes where.

---

## Source-of-Truth Precedence

When information conflicts across layers, this hierarchy resolves it:

**Tier 1 — AUTHORITATIVE (Always Wins)**
- Live instruction from an authorized human
- `decisions.md`
- `regressions.md`

**Tier 2 — OPERATIONAL (Current Truth)**
- `ROLE.md`
- `state.md` / `state/` files
- `ESCALATION.md`

**Tier 3 — CONTEXTUAL (Never Overrides Tier 1–2)**
- Shared memory / vector store
- `MEMORY.md`
- `task-notes/`
- `memory/` daily notes

**Resolution rules:**
- Higher tier always wins over lower tier.
- Within the same tier, the more recently updated source wins.
- When you detect a conflict between tiers, flag it explicitly. Don't silently pick one.

**When helping the user, verify:**
- Every file in the system maps to exactly one tier.
- There's a clear ownership model for who can update each tier.
- The agent knows how to detect and surface discrepancies.

---

## The Regression System

Regressions turn mistakes into permanent rules. This is how agents learn.

### Format

```markdown
## Regressions

| ID | Date | What Happened | Rule | Severity |
|----|------|---------------|------|----------|
| REG-001 | 2026-03-15 | Agent sent credentials in a Slack message | NEVER include plaintext credentials in messages, files, or logs. Use env var references only. | CRITICAL |
| REG-002 | 2026-03-18 | Dashboard showed stale data for 3 days | State files must be refreshed daily. Any state file >48h old triggers a maintenance flag. | HIGH |
| REG-003 | 2026-03-22 | Agent repeated a question the user already answered | If the user said it, it's captured. Search memory before asking. | MEDIUM |
```

### Rules for regressions

1. **Created immediately** when a mistake is identified. Not after analysis. Not tomorrow. Now.
2. **Loaded every session.** The agent re-reads all regressions on boot.
3. **Never deleted.** Severity can be downgraded. Entries can be marked resolved. But they stay in the file.
4. **Any agent can add.** Only the system owner can modify or remove.
5. **Severity levels:** CRITICAL (caused real damage), HIGH (significant risk), MEDIUM (quality issue), LOW (minor friction).
6. **Review quarterly** — some regressions may be obsolete if the underlying system changed.

**When helping the user implement this:**
- Start with an empty regressions file with the header structure.
- Coach them: "What are the top 3 mistakes your agent keeps making?" — those become the first entries.
- Set up the boot sequence so regressions load in Phase 3 (Rules), before any work begins.

---

## The Decision Log

Decisions get relitigated when there's no record of why they were made. The decision log kills this.

### Format

```markdown
## Decisions

| ID | Date | Decision | Context | Decided By | Status |
|----|------|----------|---------|------------|--------|
| DEC-001 | 2026-03-10 | Use Supabase as CDP, not Airtable | Need RLS, real-time subscriptions, and SQL. Airtable doesn't scale past 50K records. | Jaydyn | ACTIVE |
| DEC-002 | 2026-03-14 | Morning brief sent at 6:30am, not 7:00am | User checks phone at 6:30. By 7:00 they're already in meetings. | Calvin | ACTIVE |
| DEC-003 | 2026-04-01 | Deprecated: Slack-only comms | Superseded by DEC-008 (multi-channel). | Jaydyn | DEPRECATED |
```

### Rules for decisions

1. **Include the WHY.** A decision without context is trivia.
2. **Track status:** ACTIVE, DEPRECATED, SUPERSEDED (link to replacement).
3. **Decisions are Tier 1.** They override operational and contextual layers.
4. **When a new decision contradicts an old one,** mark the old one SUPERSEDED with a reference to the new one. Don't delete.
5. **Any agent can propose.** Only authorized humans confirm.

---

## Memory Maintenance Cadence

Memory that isn't maintained decays. Set these cadences and enforce them.

### Daily
- [ ] Write daily log (`memory/YYYY-MM-DD.md`) — what happened, what was decided, what's pending.
- [ ] Sync critical updates to shared memory store (vector store, Obsidian, etc.).
- [ ] Refresh `state.md` — current focus, blockers, health.
- [ ] Update `TASKQUEUE.md` — new tasks added, completed tasks marked, blocked tasks noted.

### Every 3 Days
- [ ] Review last 3 days of `memory/` files.
- [ ] Distill patterns, learnings, and key facts upward into `MEMORY.md`.
- [ ] Prune stale entries from `MEMORY.md` (anything no longer current).
- [ ] Check for orphaned task notes (tasks completed but notes not updated).

### Weekly
- [ ] Full `MEMORY.md` review — is everything current? Remove what's stale, add what's missing.
- [ ] Update all `state/` files — verify each reflects actual current truth.
- [ ] Verify shared stores are current (vector store, knowledge vault).
- [ ] Run Memory Maturity Score (see below) and log it.
- [ ] Review regressions — any that are now obsolete due to system changes?

### Quarterly
- [ ] Identity calibration — does the agent's actual behavior match IDENTITY.md?
- [ ] Full regression review — downgrade resolved, flag obsolete.
- [ ] Decision log audit — mark deprecated decisions, verify active ones still hold.
- [ ] Archive raw memory older than 90 days.

**When helping the user set this up:**
- Start with daily only. Add 3-day and weekly once daily is habitual.
- Automate what you can (e.g., cron for daily log creation, reminders for weekly review).
- The most common failure mode is skipping maintenance. Build it into the agent's boot sequence or task queue.

---

## Memory Maturity Score

Measure how healthy the memory system is. Run weekly.

### Metrics

| Metric | Calculation | Target |
|--------|-------------|--------|
| **Connectivity** | % of memory files with at least one backlink or cross-reference | > 70% |
| **Freshness** | % of state/hot files updated within the last 7 days | > 80% |
| **Orphan Rate** | % of files with zero incoming or outgoing connections | < 15% |

### Score Formula

```
Memory Maturity Score = Connectivity + Freshness - Orphan Rate
```

**Interpretation:**
- **> 135** — Excellent. Memory system is healthy and well-connected.
- **100–135** — Good. Some gaps but functional.
- **70–100** — Needs attention. Likely orphaned files and stale state.
- **< 70** — Critical. Memory system is decaying. Prioritize maintenance.

**When helping the user:**
- Build a simple script or checklist that calculates this.
- Log the score weekly in `state.md` or a dedicated `state/memory-health.md`.
- If the score drops below 100 two weeks in a row, flag it as a priority.

---

## The Dreaming Cycle

An overnight maintenance process that strengthens memory connections without making destructive changes.

### Rules
- **READ + APPEND only.** No modifications. No deletions. No overwrites.
- **Runs during off-hours** (overnight, weekends, low-activity periods).
- **Fixes happen during business hours** when a human can review.

### What the dreaming cycle does
1. **Scan for missing backlinks** — If File A references a concept in File B but doesn't link to it, log the suggested link.
2. **Detect contradictions** — If two files assert conflicting facts, log both with a `[CONTRADICTION]` tag.
3. **Tag stale notes** — Files not updated in 14+ days in the hot/working layers get tagged `[STALE]`.
4. **Suggest promotions** — Raw memory entries that appear in 3+ daily logs get flagged for promotion to MEMORY.md.
5. **Log orphans** — Files with zero connections get tagged `[ORPHAN]` for review.

### Output
All findings go into a `dreaming-log.md` file:

```markdown
## Dreaming Cycle — 2026-04-15

### Suggested Backlinks
- state/pipeline-status.md → should reference decisions.md#DEC-012
- MEMORY.md (line 45) → relates to task-notes/dashboard-rebuild.md

### Contradictions
- state.md says "pipeline sync: healthy" but memory/2026-04-14.md logged sync failures

### Stale
- state/team-roster.md — last updated 2026-03-28 (18 days)

### Promotion Candidates
- "Morning brief timing" appears in 5 daily logs — promote to MEMORY.md?

### Orphans
- task-notes/old-migration-plan.md — zero references
```

---

## Data Routing Rules

Not all data belongs in the same place. Route it correctly.

| Data Type | Destination | Examples |
|-----------|-------------|---------|
| Structured data | Database / CDP | Contacts, metrics, transactions, subscriptions, event logs |
| Context & reasoning | Vector store / shared memory | Why decisions were made, lessons learned, institutional knowledge |
| Daily activity | Local files (`memory/`) | Session logs, daily journals, raw notes |
| Current truth | State files (`state.md`, `state/`) | Agent status, team roster, pipeline health |
| Permanent rules | Rules files | Regressions, decisions |
| Identity | Identity files | Personality, role, voice |

**Anti-patterns to prevent:**
- ❌ Dumping structured data into vector stores (unqueryable, unstructured).
- ❌ Using MEMORY.md as a database (it's a dashboard, not a table).
- ❌ Storing current truth in daily logs (it gets buried and goes stale).
- ❌ Putting credentials anywhere in the memory system (use env vars or secret stores).

---

## Boot Sequence Design

The agent's boot sequence determines which files load, in what order, and under what conditions.

### Recommended phases

| Phase | What Loads | Purpose |
|-------|-----------|---------|
| 1. Identity | SOUL.md, IDENTITY.md, ROLE.md | Who am I? |
| 2. Context | USER.md, ESCALATION.md | Who do I serve? |
| 3. Rules | regressions.md, decisions.md | What are my constraints? |
| 4. State | state.md, TASKQUEUE.md | What's happening now? |
| 5. Situational | MEMORY.md, today's daily log, shared memory query | What do I need to know? |

### Boot sequence rules
- If a file doesn't exist: skip it, log the gap, continue. Never stall on a missing file.
- Load order matters — identity before rules, rules before state.
- Keep total boot payload small. If it takes more than 90 seconds to load everything, cut.
- Some files load conditionally (e.g., task notes load only when starting a specific task).
- Some files load on-demand (e.g., shared memory queries when working in a specific domain).

---

## Implementation Checklist

When helping a user build their memory system, walk through this:

### Phase 1: Foundation
- [ ] Create directory structure (`state/`, `memory/`, `task-notes/`)
- [ ] Write IDENTITY.md — who is this agent?
- [ ] Write ROLE.md — what's in scope, what's out?
- [ ] Create empty regressions.md with header structure
- [ ] Create empty decisions.md with header structure
- [ ] Create MEMORY.md with section headers (Active Initiatives, Key Metrics, Current Blockers)

### Phase 2: User Context
- [ ] Write USER.md — profile the primary humans the agent serves
- [ ] Define ESCALATION.md — what can the agent decide alone?
- [ ] Document communication preferences per user

### Phase 3: Operational
- [ ] Set up TASKQUEUE.md format
- [ ] Create first daily log template
- [ ] Define state.md structure
- [ ] Configure boot sequence (what loads when)

### Phase 4: Maintenance
- [ ] Set daily maintenance checklist
- [ ] Schedule weekly memory maturity check
- [ ] Create dreaming cycle (if running overnight processes)
- [ ] Define data routing rules for the agent's domain

### Phase 5: Hardening
- [ ] Populate first 3–5 regressions from known failure patterns
- [ ] Log first 3–5 canonical decisions
- [ ] Run first memory maturity score
- [ ] Test boot sequence end-to-end

---

## Multi-Agent Considerations

If designing memory for an agent squad:

- **SOUL.md is shared.** All agents load the same values and culture.
- **IDENTITY.md and ROLE.md are per-agent.** Each agent has its own lane.
- **Regressions and decisions can be shared or per-agent.** Shared = universal rules. Per-agent = domain-specific lessons.
- **Shared memory (vector store, database) bridges agents.** One agent's output becomes another's input.
- **Protected file tiers:** Some files only the system owner edits. Some any agent can append to. Define this upfront.
- **Sub-agents are ephemeral.** They don't get memory systems. They get task briefs and return results.

---

*Methodology developed by WILD Success. Adapted for AI-assisted memory architecture by AI Insiders.*
