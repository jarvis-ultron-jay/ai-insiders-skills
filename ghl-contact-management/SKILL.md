---
name: ghl-contact-management
description: GoHighLevel (GHL) contact management via API. Use when working with GHL/LeadConnector contacts including (1) searching contacts by email or name, (2) viewing contact details and tags, (3) adding or removing tags to trigger automations, (4) updating contact information, (5) checking payment and contract status, (6) handling duplicate contacts and merges, (7) triggering welcome packs or onboarding automations via tag manipulation. Supports multi-location (sub-account) setups. Triggers on GHL, GoHighLevel, LeadConnector, contact lookup, tag, welcome pack, payment check, or CRM tasks.
---

# GHL Contact Management

Manage contacts in GoHighLevel (GHL) / LeadConnector via the v2 API. This skill covers contact lookups, tag management, duplicate handling, and automation triggers across multi-location setups.

## Install

```bash
curl -sL https://ai-insiders.ai/install.sh | bash -s ghl-contact-management
```

## Pairs well with

- [`ghl-pipeline-sync`](../ghl-pipeline-sync) — pull pipeline and opportunity data alongside contacts
- [`intercom-cs`](../intercom-cs) — the CS-side workflow that triggers most contact lookups
- [`supermemory-api`](../supermemory-api) — persist solved ticket answers (e.g., welcome pack fixes) for reuse

## Authentication

All requests require:
```
Authorization: Bearer <pit_token>
Version: 2021-07-28
Accept: application/json
Content-Type: application/json
```

**PIT tokens are location-specific.** Each sub-account (location) has its own Private Integration Token. Store these in your agent's `TOOLS.md` mapped to location names and IDs.

Base URL: `https://services.leadconnectorhq.com`

## Core Workflows

### 1. Search for a Contact

Search by email (most reliable):
```
GET /contacts/?locationId={location_id}&query={email}&limit=20
```

Search by name:
```
GET /contacts/?locationId={location_id}&query={name}&limit=20
```

**Important:** If the contact could be in multiple sub-accounts (e.g., enrolled in multiple programs), search each relevant location. A contact may exist in one sub-account but not another.

### 2. Get Contact Details

```
GET /contacts/{contact_id}?locationId={location_id}
```

Key fields to check:
- `tags` — current tags on the contact
- `email` — primary email
- `additionalEmails` — secondary emails
- `customFields` — may contain contract dates, program info, payment status
- `dateAdded` — when they entered the system

### 3. Add Tags

```
POST /contacts/{contact_id}/tags
Body: {"tags": ["tag_name"]}
```

Tags are the primary mechanism for triggering GHL automations. Common use cases:
- Adding a purchase tag triggers a welcome pack email sequence
- Adding a certification tag marks program completion
- Adding a "Push to Circle" tag triggers community platform invitations

### 4. Remove Tags

```
DELETE /contacts/{contact_id}/tags
Body: {"tags": ["tag_name"]}
```

### 5. Update Contact Information

```
PUT /contacts/{contact_id}
Body: {
 "locationId": "{location_id}",
 "email": "updated@example.com",
 "firstName": "Updated",
 "lastName": "Name"
}
```

## Welcome Pack Troubleshooting

When a customer reports they never received their welcome pack / onboarding emails:

### Diagnostic Steps

1. **Search the contact** in the relevant sub-account
2. **Check for duplicate accounts** — search by email AND name. Duplicates are a common cause of welcome packs not firing
3. **Check tags** — is the purchase/enrollment tag present?
4. **Check automation history** if accessible

### Fix: Trigger Welcome Pack

If the welcome pack was **never sent** (automation never fired):
1. Remove the purchase/enrollment tag
2. Wait 2–3 seconds
3. Re-add the same tag

This re-triggers the automation tied to that tag. Only works if the welcome pack was never sent in the first place.

**If the welcome pack was previously sent** but the customer lost access, the remove/re-add trick won't work. Instead, manually send the relevant onboarding information or trigger a different automation.

### Fix: Duplicate Contacts

If duplicate contacts exist:
1. Identify which account has the contract/payment history (this is the primary)
2. Merge accounts, selecting the contract account as the primary
3. After merge, verify tags are correct
4. Trigger the welcome pack if needed using the remove/re-add method

## Multi-Location Strategy

In a multi-location GHL setup (e.g., different programs in different sub-accounts):

- **Always identify which program** the customer is asking about before searching
- **Search the correct sub-account** for that program
- **A customer may exist in multiple locations** with different data in each
- **Tags are location-specific** — a tag in one sub-account doesn't appear in another
- **Cross-reference when needed** — if a customer mentions multiple programs, check each relevant location

## Common Tag Patterns

| Tag Pattern | Purpose |
|-------------|---------|
| Program purchase tags | Trigger welcome pack / onboarding automation |
| Certification tags | Mark program completion |
| Platform push tags | Trigger invitations to community platforms |
| Payment status tags | Track payment plan status |
| Cancellation flags | Flag accounts for cancellation review |

Exact tag names are organisation-specific. Document your tag conventions in `TOOLS.md`.

## Contact Search Tips

- **Email search is most reliable.** Name searches can return partial matches.
- **Check alternate emails.** Customers often use different emails for different things. The `additionalEmails` field may have their other addresses.
- **URL-encode search queries.** Special characters in emails (like `+`) need encoding.
- **Pagination:** Default limit is 20. Use `limit` and `startAfterId` for larger result sets.

## Rate Limits

GHL API allows approximately 100 requests per minute per location. When batch processing:
- Add 500ms–1s delays between requests
- Respect 429 responses and back off
- Serialise operations within a single location
- Parallel requests across different locations are generally safe

## Error Handling

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | Success | Process response |
| 400 | Bad request | Check request body format |
| 401 | Unauthorised | Verify PIT token and location ID match |
| 404 | Not found | Contact doesn't exist in this location |
| 422 | Validation error | Check field values and required fields |
| 429 | Rate limited | Wait and retry with backoff |

## Integration Notes

GHL automations triggered by tags run asynchronously. After adding a tag:
- The automation may take 30 seconds to several minutes to execute
- Email deliverability depends on the customer's email provider
- Check automation logs in GHL if the customer reports not receiving expected emails
- Consider sending manual fallback communications for critical onboarding steps
