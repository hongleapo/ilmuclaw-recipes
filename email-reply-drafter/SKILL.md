---
name: openclaw-email-reply-drafter
description: "Use this skill when the user wants ILMU Claw to draft a reply to a specific email or thread. Triggers include: 'draft a reply to', 'write a response to this email', 'help me reply to', 'draft a follow-up to [person]', or any request to respond to a single known thread. On-demand only — not scheduled. Best suited for single-thread replies, not bulk inbox triage."
---

# ILMU Claw — Email Reply Drafter

**Status:** CORE — on-demand only

---

## What this recipe does

1. Reads the specified email thread (by sender, subject, or keyword)
2. Infers the appropriate response — what the sender needs, what tone fits
3. Drafts a reply in the user's voice
4. Optionally produces two variants: short (3–5 lines) and detailed (full response)
5. Saves to Drafts — never auto-sends
6. Shows the draft inline before saving so the user can approve or edit

---

## Tools / connectors required

| Connector | Purpose |
|-----------|---------|
| Gmail | Read thread, save draft reply |

---

## Structured output schema

```json
{
  "thread_id": "thread_abc123",
  "from": "sender@example.com",
  "subject": "Re: Project update",
  "thread_summary": "Sender asked for a status update on the Q3 deliverable by end of week.",
  "inferred_intent": "They need a progress confirmation and an ETA.",
  "variants": {
    "short": "Hi [Name], all on track — will have it to you by Friday EOD.",
    "detailed": "Hi [Name], thanks for checking in. The Q3 deliverable is progressing well — I'm targeting Friday EOD for delivery. Happy to jump on a call if you need an earlier preview."
  },
  "selected_variant": "short",
  "draft_saved": false,
  "tone_notes": "Matched from 3 recent sent emails — concise, no filler."
}
```

---

## Ready-to-run prompt (copy/paste into ILMU Claw)

```
Find the latest email from [person/topic] that needs a reply.

1. Summarise the thread in one line.
2. Infer what they need from me.
3. Draft two reply variants — SHORT (3–5 lines) and DETAILED (full response) — in my usual tone.
4. Show me both variants before saving.
5. Save the one I choose to my Drafts. Do NOT send.

Return as JSON: { "thread_id", "from", "subject", "thread_summary", "inferred_intent", "variants": { "short", "detailed" }, "selected_variant", "draft_saved", "tone_notes" }
```

---

## Execution rules for ILMU Claw

1. **Never auto-send.** Always save to Drafts. Confirm with user before saving.
2. **Always show before saving.** Surface both variants inline; wait for selection.
3. **Tone scan required.** Read at least 3 recent sent emails from the user before drafting to calibrate voice.
4. **Thread context.** Read the full thread, not just the last message. Earlier context affects the right reply.
5. **Two variants always.** Even if one seems obviously right, produce both. The user decides.
6. **If thread not found.** Return an error object: `{ "error": "Thread not found", "searched_for": "[query]" }`. Do not guess.

---

## Failure handling

| Failure | Behaviour |
|---------|-----------|
| Thread not found | Return error JSON, ask user to clarify sender or subject |
| Gmail auth error | Abort and notify user in-chat |
| Draft save fails | Show draft inline with copy instruction; set `draft_saved: false` |
| Tone scan fails (no sent mail) | Proceed with neutral professional tone; note this in `tone_notes` |

---

## Example output (inline before saving)

```
Thread: Shannon C — "Housing confirmation"
Summary: Shannon confirmed the lease is signed; needs your counter-signature by Friday.

--- VARIANT A (SHORT) ---
Hi Shannon, thanks for confirming — I'll get this signed and back to you by Thursday.

--- VARIANT B (DETAILED) ---
Hi Shannon, great news on the lease. I'll sign and return it to you by Thursday EOD — 
let me know if you need it sooner and I'll prioritise.

Which variant should I save to Drafts? (A / B / neither)
```
