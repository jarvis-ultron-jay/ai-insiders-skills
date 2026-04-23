---
name: cancellation-handling
description: Handle customer cancellation requests for subscription or contract-based programs. Use when a customer asks to cancel, withdraw, exit, or end their enrollment. Covers the full workflow including policy enforcement, empathetic conversation techniques, financial hardship handling, deferral alternatives, escalation paths, and the self-harm safety protocol. Also applies to refund requests, cooling-off period questions, and contract disputes. Triggers on cancellation, cancel, withdraw, refund, exit program, end subscription, hardship, deferral, or cooling-off tasks.
---

# Cancellation Handling

Handle cancellation requests for contract-based programs and subscriptions. This skill balances firm policy enforcement with genuine empathy. The goal is retention where possible, clear and kind communication always, and immediate escalation when safety is at risk.

> ### 🛑 SAFETY FIRST — READ BEFORE ANYTHING ELSE
>
> **If a customer mentions self-harm, suicide, or threatens to harm themselves, skip everything below and jump directly to [Self-Harm / Suicide Protocol](#self-harm--suicide-protocol).**
>
> Human safety overrides all other procedures. No questions. No retention attempt. Cancel the account, escalate to management, and provide crisis resources immediately.

## Install

```bash
curl -sL https://ai-insiders.ai/install.sh | bash -s cancellation-handling
```

## Pairs well with

- [`intercom-cs`](../intercom-cs) — handle the conversation, tag it, and escalate per this playbook
- [`ghl-contact-management`](../ghl-contact-management) — look up contract status and payment history
- [`humanizer`](../humanizer) — write responses that don't sound robotic or hollow

## Core Principle

The answer to cancellation requests outside the cooling-off period is **no**. But HOW you say no is everything. Customers are people going through real situations. Be kind, be clear, and never be cruel.

## Decision Tree

```
Customer requests cancellation
├── Within cooling-off period?
│ ├── YES → Process cancellation per policy
│ └── NO → Continue below
│
├── Mentions self-harm or suicide?
│ ├── YES → IMMEDIATE: Safety Protocol (see below)
│ └── NO → Continue below
│
├── First contact about cancellation?
│ ├── YES → Explore their situation
│ └── NO → Review previous context, adjust approach
│
├── Financial hardship?
│ ├── YES → Explore payment plan alternatives
│ └── NO → Continue below
│
├── Program dissatisfaction?
│ ├── YES → Understand specific issues, offer support
│ └── NO → Continue below
│
├── Life circumstances (health, family, work)?
│ ├── YES → Explore deferral options
│ └── NO → Address their specific concern
│
└── Aggressive / demanding?
 └── Hold firm, stay kind, escalate if needed
```

## Self-Harm / Suicide Protocol

**This overrides ALL other procedures. Human safety comes first.**

If a customer mentions self-harm, suicide, or threatens to harm themselves:

1. **Immediately respond:**
 "I'm so sorry you're going through this. Your wellbeing matters more than anything else. We're going to take care of your account right now."

2. **Confirm cancellation.** No questions, no retention attempt. Just do it.

3. **Immediately escalate to management** with: customer name, conversation link, what was said.

4. **Do NOT:**
 - Engage in counselling
 - Ask probing questions about their mental state
 - Delay or hesitate
 - Apply normal cancellation policy

5. **Provide helpline numbers if appropriate.** Use country-appropriate crisis lines based on the customer's location. Common examples:
 - National crisis hotlines
 - Mental health support services
 - Emergency services (if immediate danger)

6. **Priority: Human safety over contracts, revenue, policy, and everything else.**

## Conversation Approach

### Step 1: Acknowledge and Explore

Don't jump straight to policy. Find out what's going on:

- "I'm sorry to hear that. Can I ask what's prompted this?"
- "Before we go through the process, I'd love to understand what's happening for you."
- "Is there something specific about the program that isn't working?"

### Step 2: Listen and Identify the Real Issue

Common underlying reasons:
- **Financial stress** → Payment plan adjustment or pause may help
- **Overwhelmed** → Reassure them about pacing, point to replays/support
- **Not seeing value** → Help them engage with the right parts of the program
- **Life event** → Deferral to a future cohort
- **Buyer's remorse** → Reconnect them with why they signed up

### Step 3: Offer Alternatives (where applicable)

Before enforcing the contract, explore options:

| Situation | Possible Alternative |
|-----------|---------------------|
| Financial hardship | Adjusted payment plan, temporary pause |
| Overwhelmed | Reduced pace, replay access, 1-on-1 support |
| Wrong program fit | Transfer to a more suitable program |
| Life event / timing | Deferral to next cohort |
| Not engaging | Accountability buddy, check-in schedule |

### Step 4: Enforce Policy (when needed)

If no alternative works and the customer is outside cooling-off:

- Be direct but kind
- Reference the contract they signed
- Don't apologise for the policy itself
- Acknowledge their frustration without using hollow empathy phrases

**Do:**
"I hear you, and I know this isn't what you want to hear. The contract you signed is binding outside the cooling-off window. What I can do is look at what's going on for you and see if there's a way we can make the program work better."

**Don't:**
"Unfortunately, as per our terms and conditions, cancellations outside the cooling-off period are not permitted."

**Don't:**
"I understand your frustration." (Empty phrase. Show you understand through action.)

### Step 5: Escalate When Necessary

Escalate to a human manager when:
- Customer is threatening legal action
- Customer is in genuine financial crisis
- The situation involves a formal complaint
- The conversation has gone through multiple rounds with no resolution
- You are unsure about the right approach
- Any safety concern whatsoever

## Hard Rules

1. **Never suggest ways to cancel.** You work for the company. Don't give customers ideas about loopholes, chargebacks, or workarounds.
2. **Never reveal cancellation loopholes.** If exceptions exist for extreme cases, those are management decisions, not yours to offer.
3. **Never guess on policy.** If you're not sure whether an exception can be made, escalate. A wrong answer is worse than a delayed one.
4. **Never make promises you can't keep.** "Let me check with my team" is always better than committing to something you're not authorised to deliver.
5. **Never dismiss emotions.** Even if the request is clearly against policy, the person's feelings are real.
6. **Never rush the conversation.** Take the time to understand their situation properly.

## Tone Guide

| Customer Energy | Your Approach |
|----------------|---------------|
| Calm, straightforward | Direct and efficient. Respect their time. |
| Emotional, upset | Warm, patient. Acknowledge before responding. |
| Angry, aggressive | Stay calm, don't match their energy. Be kind but firm. |
| Manipulative, pushing | Hold your ground. Don't fold on policy. Don't argue. |
| Desperate, struggling | Maximum empathy. Explore every alternative before saying no. |

## Cooling-Off Period

The cooling-off period is defined in the customer's contract. It is typically a set number of days from the date of signing. During this window:
- Cancellation requests should be processed according to organisational policy
- Verify the contract signing date before confirming eligibility
- If borderline (e.g., customer says they're within the period but timing is unclear), escalate for verification

**Setup note:** If your organisation doesn't have a defined cooling-off period, configure it in your agent's `TOOLS.md` before using this skill. Example:

```
cooling_off_days: 14
cooling_off_policy_link: https://example.com/terms
```

## Documentation

After handling a cancellation request:
1. Tag the conversation appropriately (e.g., "Cancellation Request", "Financial Hardship", "Deferral Request")
2. Leave an internal note summarising: what the customer asked, what was offered, what was decided
3. If escalated, include full context so the next person doesn't have to re-ask
4. Log the outcome for pattern tracking (helps identify program issues)

## Deferral Process

When a deferral is the right option:
1. Confirm deferral availability with the team (not all programs offer it)
2. Explain what deferral means: they pause now and join a future cohort
3. Clarify what happens to their payments during deferral
4. Get written confirmation from the customer
5. Process via the appropriate internal channel
6. Follow up to confirm the deferral is set up

## Financial Hardship

When a customer cites genuine financial hardship:
1. Express empathy through action, not phrases
2. Check if adjusted payment plans are available
3. Explore if a payment pause is possible
4. Escalate to finance/management with a recommendation
5. Never promise a specific outcome before getting approval
6. Document the hardship reason for internal records
