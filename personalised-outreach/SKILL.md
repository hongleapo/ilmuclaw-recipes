---
name: openclaw-personalised-outreach
description: "Use this skill when the user wants ILMU Claw to run personalised cold or marketing email outreach, A/B test subject lines or copy, track replies, or chase follow-ups automatically. Triggers include: 'send outreach to my leads list', 'write personalised cold emails for', 'set up A/B email test', 'follow up on people who haven't replied', 'track my outreach campaign'. Covers both launch (on-demand) and follow-up monitoring (scheduled daily)."
---

# ILMU Claw — Personalised Outreach + Follow-up Engine

**Status:** SCHEDULABLE — launch on demand; follow-up check runs daily

---

## What this recipe does

1. Ingests a lead list (pasted directly into chat, or linked from a Google Drive file)
2. For each contact: researches personalisation hooks via web search (company news, role, context)
3. Writes a personalised email — two A/B variants (different subject line + opening)
4. Logs each contact, variant assigned, and send status to a Google Drive tracking document
5. Monitors daily for replies and no-reply timeouts (default: 3 days)
6. Notifies the user in-chat when someone replies or is due a follow-up
7. **People leads own the send** — ILMU Claw drafts; a human reviews and sends

---

## Tools / connectors required

| Connector | Purpose |
|-----------|---------|
| Gmail | Draft and send outreach (human-approved) |
| Google Drive | Campaign tracking log (stored as a document) |
| Web search | Lead research + personalisation hooks |

---

## Structured output schema

### Per-contact record (returned in JSON)

```json
{
  "campaign_id": "camp_001",
  "run_date": "YYYY-MM-DD",
  "contacts": [
    {
      "id": "contact_001",
      "name": "Alex Tan",
      "email": "alex@example.com",
      "company": "Acme Corp",
      "personalisation_hook": "Acme just raised a Series B; Alex is leading their new AI team.",
      "variant": "A",
      "subject": "Congrats on the raise — quick thought on AI tooling",
      "body": "Hi Alex, saw the Series B news — congrats...",
      "status": "DRAFT",
      "sent_at": null,
      "replied_at": null,
      "follow_up_due": "YYYY-MM-DD",
      "notes": ""
    }
  ],
  "ab_test": {
    "variant_a_subject": "Congrats on the raise — quick thought on AI tooling",
    "variant_b_subject": "One idea for your new AI team at Acme"
  },
  "summary": {
    "total_contacts": 10,
    "drafts_ready": 10,
    "sent": 0,
    "replied": 0,
    "follow_up_due_today": 0
  }
}
```

### Status values

| Status | Meaning |
|--------|---------|
| `DRAFT` | Written, not yet sent |
| `SENT` | Approved and sent by human |
| `REPLIED` | Reply received |
| `FOLLOW_UP_DUE` | No reply after threshold (default 3 days) |
| `CLOSED` | No further action needed |

---

## Ready-to-run prompt (copy/paste into ILMU Claw)

### Launch batch

```
From this list of leads [paste list or share a Google Drive file link]:

1. Research each contact via web search — find one personalisation hook per person (recent news, role change, shared context).
2. Write a personalised outreach email for each with TWO A/B variants:
   - Variant A: [describe angle A, e.g. "lead with their company news"]
   - Variant B: [describe angle B, e.g. "lead with the value prop directly"]
3. Log every contact to a new Google Drive document: Name, Email, Company, Variant, Subject, Status, Sent At, Replied At, Follow-up Due.
4. Return the full campaign JSON before saving anything.
5. Do NOT send any emails. Save to Gmail Drafts only — I will review and send.
```

### Daily follow-up check (scheduled)

```
Check my outreach tracking document in Google Drive [filename or link].
1. Flag anyone with status SENT and no reply after 3 days — update their status to FOLLOW_UP_DUE.
2. Draft a short follow-up email for each FOLLOW_UP_DUE contact.
3. Send me a summary in chat: who replied, who needs a follow-up, who to close.
Return as JSON before sending the summary.
```

---

## Scheduled variant (daily follow-up monitor)

```
Schedule: daily 09:00 (local time)
Prompt: [daily follow-up check prompt above]
Drive doc: [your campaign tracking document name or link]
Follow-up threshold: 3 days (adjustable)
```

---

## Execution rules for ILMU Claw

1. **Human owns the send.** Never call send on outreach emails. Draft only — user reviews in Gmail and sends manually.
2. **JSON before side-effects.** Return the full campaign JSON before writing to Drive or saving drafts.
3. **Personalisation is required.** Each email must reference at least one contact-specific detail. Generic emails are not acceptable output.
4. **A/B must differ meaningfully.** Variants must differ in subject line AND opening sentence. Changing one word is not an A/B test.
5. **Drive doc is the source of truth.** Always read the tracking document before drafting follow-ups — do not re-draft for contacts already marked REPLIED or CLOSED.
6. **Idempotent log writes.** Check for an existing entry by email before inserting. Update, don't duplicate.

---

## Failure handling

| Failure | Behaviour |
|---------|-----------|
| Lead list empty or unparseable | Return error JSON, ask user to re-paste or re-share the Drive file |
| Personalisation research fails for a contact | Set `personalisation_hook: null`, flag in notes, proceed with generic hook |
| Drive write fails | Return full JSON in-chat so user can log manually |
| Gmail auth error | Abort batch, notify user in-chat |

---

## Example follow-up summary (sent in chat)

```
📤 Outreach Follow-up — 23 Jun 2026

✅ REPLIED (2)
• Alex Tan (Acme Corp) — replied yesterday
• Priya K (Vertex AI) — replied this morning

⏰ FOLLOW-UP DUE (3) — drafts saved
• Jordan L (TechCo) — no reply after 4 days
• Sam R (Horizon) — no reply after 3 days
• Casey M (Bloom) — no reply after 3 days

🚫 CLOSED (1)
• Morgan B — unsubscribed
```
