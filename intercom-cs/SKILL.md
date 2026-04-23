---
name: intercom-cs
description: Customer service operations in Intercom. Use when handling support conversations including (1) searching and retrieving conversations, (2) drafting replies as internal notes for human review, (3) tagging and categorising conversations, (4) assigning conversations to teams or admins, (5) triaging inboxes by priority, (6) closing resolved conversations, (7) polling for new or updated conversations. Covers the full CS agent workflow from inbox monitoring through to resolution. Triggers on Intercom, support ticket, triage, inbox, conversation, internal note, or customer service tasks.
---

# Intercom CS Operations

Operate as a customer service agent inside Intercom. This skill covers the full workflow: monitoring the inbox, triaging conversations, drafting responses, and managing conversation lifecycle.

## Install

```bash
curl -sL https://ai-insiders.ai/install.sh | bash -s intercom-cs
```

## Pairs well with

- [`cancellation-handling`](../cancellation-handling) — decision tree + safety protocol for cancellation requests
- [`ghl-contact-management`](../ghl-contact-management) — CRM-side context for the customer you're replying to
- [`supermemory-api`](../supermemory-api) — persistent memory for learned answers across conversations

## Authentication

All requests require:
```
Authorization: Bearer <api_token>
Content-Type: application/json
Intercom-Version: 2.11
```

Store the API token in your agent's `TOOLS.md` or environment config. Never hardcode tokens in skill files.

The Intercom API version evolves — check the [Intercom API changelog](https://developers.intercom.com/docs/changelog) and bump the version header when older versions deprecate.

## Core Workflows

### 1. Monitor the Inbox

Poll for recent conversations sorted by last update:

```
GET /conversations?order=desc&sort_by=updated_at&per_page=20
```

For targeted monitoring, use search:

```
POST /conversations/search
Body: {
 "query": {
 "operator": "AND",
 "value": [
 {"field": "open", "operator": "=", "value": true},
 {"field": "statistics.last_contact_reply_at", "operator": ">", "value": <unix_timestamp>}
 ]
 },
 "pagination": {"per_page": 50}
}
```

Useful search fields:
- `open` — filter open/closed
- `statistics.first_admin_reply_at` — null means customer never got a reply
- `waiting_since` — non-null means customer is waiting
- `admin_assignee_id` — filter by assigned admin
- `team_assignee_id` — filter by team inbox
- `tag_id` — filter by tag

### 2. Read a Conversation

```
GET /conversations/{conversation_id}
```

Check key fields:
- `source.subject` and `source.body` — original message
- `source.author.email` and `source.author.name` — who sent it
- `tags.tags[]` — applied tags
- `statistics.first_admin_reply_at` — null = never replied
- `waiting_since` — non-null = customer waiting for response
- `state` — open/closed/snoozed
- `conversation_parts` — full thread (use `GET /conversations/{id}?display_as=plaintext` for cleaner output)

### 3. Draft a Response (Internal Note)

During training or review phases, post responses as internal notes rather than customer-facing replies:

```
POST /conversations/{conversation_id}/reply
Body: {
 "message_type": "note",
 "type": "admin",
 "admin_id": "<your_admin_id>",
 "body": "<p>Draft response for review:</p><p>Your drafted reply here</p>"
}
```

**To send a customer-facing reply** (only when authorised):

```
POST /conversations/{conversation_id}/reply
Body: {
 "message_type": "comment",
 "type": "admin",
 "admin_id": "<your_admin_id>",
 "body": "<p>Your reply to the customer</p>"
}
```

### 4. Tag a Conversation

**Add a tag:**
```
POST /tags
Body: {
 "name": "Tag Name",
 "id": "<conversation_id>"
}
```

Note: Use the tag name, not the tag ID. Intercom will match or create the tag.

**List available tags:**
```
GET /tags
```

### 5. Assign a Conversation

**Assign to a team inbox:**
```
POST /conversations/{conversation_id}/parts
Body: {
 "message_type": "assignment",
 "type": "admin",
 "admin_id": "<your_admin_id>",
 "assignee_id": "<team_id>"
}
```

> ### ⚠️ CRITICAL: Never include a `body` field in assignment calls.
>
> If you include body text, it gets sent to the customer as a visible message. Assignment should be a silent operation. Always omit the body field or set it to an empty string.

**Assign to an individual admin:**
```
POST /conversations/{conversation_id}/parts
Body: {
 "message_type": "assignment",
 "type": "admin",
 "admin_id": "<your_admin_id>",
 "assignee_id": "<target_admin_id>"
}
```

### 6. Close a Conversation

```
POST /conversations/{conversation_id}/parts
Body: {
 "message_type": "close",
 "type": "admin",
 "admin_id": "<your_admin_id>"
}
```

### 7. Snooze a Conversation

```
POST /conversations/{conversation_id}/parts
Body: {
 "message_type": "snooze",
 "type": "admin",
 "admin_id": "<your_admin_id>",
 "snoozed_until": <unix_timestamp>
}
```

## Triage Priority Order

When processing a backlog, prioritise in this order:

1. **Self-harm / safety concerns** — Immediate action, cancel account, escalate to humans. See [`cancellation-handling`](../cancellation-handling) for the full safety protocol.
2. **Formal complaints** — High priority, escalate with full context
3. **Financial hardship** — Sensitive handling required, escalate to appropriate team
4. **Cancellation requests** — Follow the [`cancellation-handling`](../cancellation-handling) skill's decision tree
5. **Access issues** (login, Circle, Zoom) — Usually quick to resolve
6. **Payment/billing queries** — May need finance team involvement
7. **General questions** (replays, certificates, schedules) — Standard responses
8. **Positive messages / thank-yous** — Acknowledge and close

## Inbox Audit

To assess inbox health, run these searches:

**Total open conversations:**
```json
{"query": {"field": "open", "operator": "=", "value": true}}
```

**Never replied to (waiting customers):**
```json
{
 "query": {
 "operator": "AND",
 "value": [
 {"field": "open", "operator": "=", "value": true},
 {"field": "statistics.first_admin_reply_at", "operator": "=", "value": null}
 ]
 }
}
```

**Unassigned conversations:**
```json
{
 "query": {
 "operator": "AND",
 "value": [
 {"field": "open", "operator": "=", "value": true},
 {"field": "admin_assignee_id", "operator": "=", "value": "0"}
 ]
 }
}
```

## System Noise Filtering

These email patterns are typically automated system notifications, not customer inquiries:
- `noreply@`, `no-reply@`, `notifications@`
- Addresses from notification services (payment processors, community platforms, CRM systems)

Before auto-closing, always verify the conversation content. Some system emails contain actionable customer information.

## Polling Schedule

For automated inbox monitoring, poll every 10–30 minutes:
1. Search for new conversations since last poll timestamp
2. Search for conversations where customer replied since last poll
3. Process new items through triage priority order
4. Draft responses as internal notes
5. Update last poll timestamp

Store polling state (last check timestamp) in a persistent file so it survives session restarts.

## Response Drafting Guidelines

When drafting customer responses:
- Read the full conversation thread before responding
- Check tags for context on the issue type
- Check if other admins have left internal notes
- Match the customer's tone and formality level
- Be specific about what you can and can't do
- If unsure, escalate rather than guess
- Never promise something without confirming you can deliver it

## Key API Entities

**List admins (team members):**
```
GET /admins
```

**List teams (inboxes):**
```
GET /teams
```

**Search contacts:**
```
POST /contacts/search
Body: {
 "query": {"field": "email", "operator": "=", "value": "customer@example.com"}
}
```

## Rate Limits

Intercom API has rate limits. If you receive a 429 response:
- Read the `Retry-After` header
- Wait that many seconds before retrying
- When batch processing, add 200–500ms delays between requests
