# ILMU Claw Ready-Made Recipe Skills

Ready-to-use skill recipes for ILMU Claw users running OpenClaw-compatible workflows across inbox, outreach, pricing, and marketing tasks.

Each folder contains a `SKILL.md` with the recipe goal, required connectors, structured output schema, copy-paste prompt, execution rules, and failure handling.

---

## Recipe index

| Folder | Recipe | Status | Trigger type |
|--------|--------|--------|--------------|
| `email-reply-drafter/` | Email Reply Drafter | CORE | On-demand |
| `inbox-digest/` | Inbox Digest + Action Items | SCHEDULABLE | On-demand / daily |
| `personalised-outreach/` | Personalised Outreach + Follow-up Engine | SCHEDULABLE | On-demand / daily |
| `financial-pricing-report/` | Financial Pricing Report | SCHEDULABLE | On-demand / daily |
| `marketing-assistant/` | Marketing Assistant | CORE | On-demand |

## Status definitions

| Status | Meaning |
|--------|---------|
| **CORE** | Ready to run today — no special connector dependencies |
| **SCHEDULABLE** | Can be set on a recurring schedule (daily / weekly) |

---

## Using a recipe

1. Open the folder for the workflow you want.
2. Read the connector requirements in `SKILL.md`.
3. Copy the ready-to-run prompt into ILMU Claw, or install the folder into your OpenClaw-compatible skills directory if your runtime supports local skill loading.
4. Keep all generated drafts, messages, and reports reviewable by a human before any external send or publish action.

---

## Copy-paste install messages

Send one of these messages to your agent to install a specific skill.

### Email Reply Drafter

```text
Install this ILMU Claw skill from GitHub:
https://github.com/hongleapo/ilmuclaw-recipes/tree/main/email-reply-drafter
```

### Inbox Digest + Action Items

```text
Install this ILMU Claw skill from GitHub:
https://github.com/hongleapo/ilmuclaw-recipes/tree/main/inbox-digest
```

### Personalised Outreach + Follow-up Engine

```text
Install this ILMU Claw skill from GitHub:
https://github.com/hongleapo/ilmuclaw-recipes/tree/main/personalised-outreach
```

### Financial Pricing Report

```text
Install this ILMU Claw skill from GitHub:
https://github.com/hongleapo/ilmuclaw-recipes/tree/main/financial-pricing-report
```

### Marketing Assistant

```text
Install this ILMU Claw skill from GitHub:
https://github.com/hongleapo/ilmuclaw-recipes/tree/main/marketing-assistant
```

---

## Quick-start: recommended first builds

1. **Inbox Digest** — highest value, lowest risk. Set it to run daily at 8am. Pairs immediately with Email Reply Drafter for reply drafting.
2. **Personalised Outreach** — higher setup because it needs a lead list and tracking document, but useful for pipeline work.
3. **Marketing Assistant** — fast on-demand content generation with optional research.

---

## Connector dependency map

```
Gmail        ──── Email Reply Drafter, Inbox Digest, Personalised Outreach, Marketing Assistant optional
Google Drive ──── Personalised Outreach, Marketing Assistant optional
Web search   ──── Personalised Outreach, Financial Pricing Report, Marketing Assistant
```

---

## Cross-recipe workflows

### Morning stack
Run Inbox Digest and Financial Pricing Report together every morning. One chat message can cover inbox triage and your market or price watchlist.

### Outreach pipeline
Marketing Assistant generates campaign copy → Personalised Outreach personalises outreach drafts at scale → Email Reply Drafter handles inbound replies.

### Inbox loop
Inbox Digest finds reply-worthy threads and drafts quick responses. Email Reply Drafter can then refine a specific thread before saving a final draft.
