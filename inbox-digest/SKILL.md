---
name: openclaw-inbox-digest
description: "Use this skill when the user wants ILMU Claw to run an email triage, inbox digest, or action-item extraction workflow. Triggers include: 'summarise my emails', 'what do I need to act on today', 'inbox digest', 'draft replies for unread emails', 'triage my inbox', or any request to process email into structured output. Covers both on-demand runs and scheduled (daily morning) configurations."
---

# ILMU Claw — Inbox Digest + Action Items

**Status:** SCHEDULABLE — runs on demand or on a daily schedule (e.g. every morning at 8am)

---

## What this recipe does

1. Scans unread / important email from the last 24 hours (or a window you specify)
2. Groups emails by sender / topic
3. For each thread: produces a one-line summary, an action label, and an urgency signal
4. Pre-drafts replies for emails that need a quick response
5. Delivers the digest as a message in your chat channel and saves drafts to Gmail

The output is **structured** (JSON → rendered as a readable summary), never freeform prose dumped into chat.

---

## Tools / connectors required

| Connector | Purpose |
|-----------|---------|
| Gmail | Read inbox, save drafts |

Delivery of the digest is sent back through your configured messaging channel (e.g. Telegram).

---

## Structured output schema

```json
{
  "digest_date": "YYYY-MM-DD",
  "window_hours": 24,
  "total_unread": 12,
  "items": [
    {
      "id": "thread_001",
      "from": "shannon@example.com",
      "subject": "Housing confirmation",
      "one_line_summary": "Shannon confirmed the lease is signed; asks for a counter-signature by Friday.",
      "action": "REPLY_NEEDED",
      "urgency": "HIGH",
      "draft_reply": "Hi Shannon, thanks for confirming — I'll get this signed and back to you by Thursday.",
      "draft_saved": false
    }
  ],
  "actions_summary": {
    "REPLY_NEEDED": 3,
    "FYI_ONLY": 5,
    "WAITING_ON_OTHER": 2,
    "NO_ACTION": 2
  }
}
```

### Action labels

| Label | Meaning |
|-------|---------|
| `REPLY_NEEDED` | You must respond; draft pre-generated |
| `FYI_ONLY` | Read and archive; no reply needed |
| `WAITING_ON_OTHER` | You've replied; waiting for their response |
| `NO_ACTION` | Newsletter, notification, or noise |

### Urgency levels

| Level | Criteria |
|-------|----------|
| `HIGH` | Deadline within 24h, or explicit request for urgency |
| `MEDIUM` | Response expected within a few days |
| `LOW` | No time pressure |

---

## Ready-to-run prompt (copy/paste into ILMU Claw)

```
Summarise my unread emails from the last 24 hours.

For each email or thread:
1. Give a one-line summary (max 20 words).
2. Assign an action label: REPLY_NEEDED, FYI_ONLY, WAITING_ON_OTHER, or NO_ACTION.
3. Assign an urgency: HIGH, MEDIUM, or LOW.
4. If the action is REPLY_NEEDED, draft a short reply in my usual tone.

Return the full result as a JSON object matching this schema:
{ "digest_date", "window_hours", "total_unread", "items": [ { "id", "from", "subject", "one_line_summary", "action", "urgency", "draft_reply", "draft_saved" } ], "actions_summary" }

After generating the JSON:
- Save drafts for any REPLY_NEEDED items (set draft_saved: true after saving).
- Send me the formatted digest here in chat — group by urgency, HIGH first.
- Do NOT auto-send any emails. Drafts only.
```

---

## Scheduled variant (daily morning digest)

To run this automatically every weekday at 8am, use the following trigger config in ILMU Claw's scheduler:

```
Schedule: weekdays 08:00 (local time)
Prompt: [paste the ready-to-run prompt above]
Window: last 18 hours (catches overnight mail without overlap)
```

---

## Execution rules for ILMU Claw

1. **Never auto-send email.** Drafts are saved to Gmail Drafts folder only. The user reviews and sends manually.
2. **Structured output first.** Always emit the JSON before taking any side-effect (saving drafts, sending digest).
3. **Validate the schema.** If a field cannot be determined, use `null` — never omit the key.
4. **Idempotent draft saving.** Check if a draft for the same thread already exists before creating a new one. Set `draft_saved: false` if skipped.
5. **Scope the window.** Default is 24h. If the user specifies a different window, honour it exactly and reflect it in `window_hours`.
6. **Tone matching.** Draft replies must match the user's existing email voice — scan 2–3 of their recent sent emails before drafting.

---

## Failure handling

| Failure | Behaviour |
|---------|-----------|
| Gmail auth error | Abort, notify user in-chat: "Email triage failed — re-authenticate Gmail." |
| Zero unread emails | Return `total_unread: 0`, reply in chat: "Inbox clear — nothing to triage." |
| Draft save fails | Set `draft_saved: false`, surface the draft text inline so user can copy manually. |

---

## Example digest output (sent in chat)

```
📬 Inbox Digest — 23 Jun 2026
12 emails · 3 need replies

🔴 HIGH
• Shannon C — Housing confirmation → REPLY NEEDED (draft saved)
• Nic — Validation sign-off due today → REPLY NEEDED (draft saved)

🟡 MEDIUM
• CK — Hermes integration update → FYI ONLY

🟢 LOW + NO ACTION (7 emails) — archived.
```
