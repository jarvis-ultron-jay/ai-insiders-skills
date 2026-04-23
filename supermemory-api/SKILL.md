---
name: supermemory-api
description: Use the Supermemory API to build and query a persistent knowledge base for your agents. Covers (1) saving resolved answers and escalation outcomes, (2) searching past solutions before responding to new queries, (3) managing container tags to isolate knowledge per team or domain, (4) building institutional memory that compounds over time, (5) periodic review and cleanup. For the broader memory architecture (local files + Supabase + Supermemory stitched together) see `agent-memory-architecture`. Triggers on Supermemory, knowledge base, save learning, store answer, search memory, FAQ, institutional knowledge, or learned answer tasks.
---

# Supermemory API

Build and query a persistent knowledge base using the Supermemory API. Use this to store answers you've learned, solutions that worked, and institutional knowledge that should persist across sessions. Think of it as your team's shared brain.

## Install

```bash
curl -sL https://ai-insiders.ai/install.sh | bash -s supermemory-api
```

## Scope

This skill is specifically about **using the Supermemory API** — endpoints, payloads, container strategy, and write/search patterns.

For the bigger picture (how Supermemory fits alongside local files, structured databases, and daily notes in a coherent memory system), install [`agent-memory-architecture`](../agent-memory-architecture). The two skills are complementary — use both if you're building a serious agent with long-term memory.

## Pairs well with

- [`agent-memory-architecture`](../agent-memory-architecture) — the full memory system Supermemory fits into
- [`intercom-cs`](../intercom-cs) — CS workflow that benefits most from stored learnings
- [`cancellation-handling`](../cancellation-handling) — store escalation outcomes here for future reference

## Authentication

```
Authorization: Bearer <api_token>
Content-Type: application/json
```

Base URL: `https://api.supermemory.ai/v3`

Store the API token in your agent's `TOOLS.md`. Use a dedicated container tag to isolate your knowledge from other agents or projects.

## Core Operations

### 1. Save a Memory

```
POST /memories
Body: {
 "content": "Your distilled knowledge here",
 "containerTags": ["your_container"]
}
```

**What to save:**
- Resolved customer questions and the correct answer
- Escalation outcomes (what was decided and why)
- Process clarifications from team members
- Edge cases and exceptions to standard procedures
- Common mistakes and how to avoid them

**What NOT to save:**
- Raw conversation transcripts (too long, too noisy)
- Customer personal data (privacy risk)
- Temporary information (one-off situations that won't recur)
- Speculative answers that haven't been confirmed

### 2. Search Memories

```
POST /search
Body: {
 "q": "search query describing what you need",
 "containerTags": ["your_container"],
 "limit": 5
}
```

Returns matching memories ranked by relevance.

**When to search:**
- Before answering any question you're not 100% sure about
- When a customer asks something that sounds like it might have come up before
- When you encounter a scenario similar to a past escalation
- Before escalating, to check if the answer already exists

### 3. List Memories

```
GET /memories?containerTags=your_container&limit=20
```

Useful for periodic review and cleanup.

### 4. Delete a Memory

```
DELETE /memories/{memory_id}
```

Remove outdated or incorrect entries during knowledge base maintenance.

## Knowledge Capture Workflow

### After Every Resolved Escalation

When a team member provides an answer to something you escalated:

1. **Distill the answer** into a concise, reusable format
2. **Include context**: What was the question? What's the correct answer? Any exceptions?
3. **Save to Supermemory** with appropriate container tag
4. **Confirm** the save in your conversation log

Example:

```json
{
 "content": "Welcome pack not received: If the welcome pack was never sent (automation didn't fire), remove the purchase tag from the contact in GHL, wait 2-3 seconds, then re-add it. This re-triggers the automation. Only works if the pack was never sent before. If it was sent but lost, manually resend the onboarding information instead.",
 "containerTags": ["cs_knowledge"]
}
```

### After Learning a New Process

When you learn how something works through trial, error, or team guidance:

1. Write it as if explaining to a colleague who's never done it
2. Include the steps, not just the outcome
3. Note any gotchas or common mistakes
4. Save with a clear, searchable description

### Periodic Knowledge Review

Schedule regular reviews (weekly or monthly) to:
- Remove outdated entries (processes that have changed)
- Merge duplicate entries
- Update entries with new information
- Identify gaps (questions that keep coming up without stored answers)

## Writing Good Memory Entries

### Keep Entries Under 200 Words

Supermemory works best with focused, concise entries. One topic per memory.

**Good:**
```
Circle login issues: If a student can't access the community platform, first check if they have an active account by searching their email in the admin panel. If no account exists, send the community invitation link (found in the program settings). If they have an account but can't log in, direct them to use the 'Forgot Password' option. If the issue persists after a password reset, escalate to the platform admin.
```

**Bad:**
```
Today I helped Sarah who couldn't log into Circle. She emailed us at 3pm and said she'd been trying for two days. I searched for her in the admin panel and found her account was inactive. I sent her the invitation link and she got in after 20 minutes. Bonnie said we should always check the admin panel first before sending links.
```

The first example is reusable knowledge. The second is a journal entry.

### Structure for Searchability

Start entries with the **topic or question** so search can match them easily:

- "Welcome pack not received: ..."
- "Circle login troubleshooting: ..."
- "Payment plan change process: ..."
- "Certificate eligibility requirements: ..."

### Include the "Why" When It Matters

If there's a non-obvious reason behind a process, include it:

```
Assignment API calls in Intercom: Never include a body field when assigning conversations to a different inbox. The body text gets sent to the customer as a visible message. Assignment should always be a silent operation.
```

The "why" prevents future agents from making the same mistake.

## Container Strategy

Use container tags to organise knowledge by domain. The example below is CS-flavoured — adapt container names to whatever domain your agent operates in (sales, ops, finance, etc.).

| Container (example) | Content |
|---------------------|---------|
| `<team>_knowledge` | General answers and processes for that team |
| `<team>_policies` | Policies, contract terms, escalation rules |
| `<team>_technical` | Platform-specific troubleshooting |
| `<team>_escalations` | Outcomes and decisions from past escalations |

**Customer service example:** `cs_knowledge`, `cs_policies`, `cs_technical`, `cs_escalations`.

You can search across multiple containers:
```json
{
 "q": "welcome pack",
 "containerTags": ["cs_knowledge", "cs_technical"],
 "limit": 5
}
```

Or use a single container if the separation isn't needed. Start simple and split later if the knowledge base grows large.

## Integration with Agent Workflow

### Before Drafting a Response

```
1. Customer asks a question
2. Search Supermemory for similar past questions
3. If match found → use stored answer (verify it's still current)
4. If no match → draft best response, escalate if unsure
5. After resolution → save the confirmed answer to Supermemory
```

### Before Escalating

```
1. You're unsure about something
2. Search Supermemory first
3. If the answer exists → use it, skip the escalation
4. If no answer → escalate, then save the outcome
```

This creates a virtuous cycle: every escalation teaches the system something, reducing future escalations.

## Rate Limits

Supermemory API is generally lenient, but:
- Don't save memories in rapid loops (batch if possible)
- Search before save to avoid duplicates
- Keep entries concise to stay within any size limits

## Maintenance Checklist

Run through this periodically:

- [ ] Review entries older than 30 days for accuracy
- [ ] Remove entries about processes that have changed
- [ ] Merge entries that cover the same topic
- [ ] Identify frequently-asked questions without stored answers
- [ ] Check that container tags are consistent
- [ ] Verify stored answers match current company policy
