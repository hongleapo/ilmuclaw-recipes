---
name: openclaw-marketing-assistant
description: "Use this skill when the user wants ILMU Claw to generate marketing content — social posts, email newsletters, campaign copy, or one-pagers — from a brief. Triggers include: 'create a marketing campaign for', 'write social posts for', 'draft an email newsletter about', 'make a one-pager for', 'write campaign copy for [product/audience]', 'generate marketing assets'. On-demand only."
---

# ILMU Claw — Marketing Assistant

**Status:** CORE — on-demand only

---

## What this recipe does

1. Takes a brief: product, audience, tone, goal, and any brand constraints
2. Generates a full set of ready-to-edit marketing assets:
   - 3 social media posts (platform-specific formatting)
   - 1 email newsletter
   - 1 one-page summary brief
3. Optionally saves assets to Google Drive as a document
4. Can do a light web search to validate claims, gather supporting stats, or check competitor messaging

---

## Tools / connectors required

| Connector | Purpose |
|-----------|---------|
| Web search | Research supporting stats, competitor context |
| Google Drive *(optional)* | Save generated assets as a document |
| Gmail *(optional)* | Draft the newsletter as a send-ready email |

---

## Structured output schema

```json
{
  "brief": {
    "product": "One Click Claw",
    "audience": "SME business owners in Malaysia",
    "tone": "Confident, plain-spoken, no jargon",
    "goal": "Drive sign-ups for the free trial",
    "brand_notes": "No em-dashes. Short sentences. Concrete benefits over adjectives."
  },
  "assets": {
    "social_posts": [
      {
        "platform": "LinkedIn",
        "character_count": 280,
        "text": "Your inbox shouldn't run your day. One Click Claw triages your email, drafts replies, and tells you what actually matters — every morning, automatically.",
        "hashtags": ["#automation", "#productivity", "#AI"],
        "cta": "Try it free →"
      },
      {
        "platform": "Twitter/X",
        "character_count": 240,
        "text": "What if your inbox was handled before you even opened your laptop? That's what we built.",
        "hashtags": ["#buildinpublic", "#AI"],
        "cta": "Link in bio"
      },
      {
        "platform": "Telegram broadcast",
        "character_count": 160,
        "text": "Morning brief: 12 emails, 3 need replies — all drafted. That's One Click Claw. Free trial open now.",
        "hashtags": [],
        "cta": "Reply YES to get access"
      }
    ],
    "email_newsletter": {
      "subject": "Your inbox, handled",
      "preview_text": "What if you opened Telegram first thing — not email?",
      "body": "Hi [Name],\n\nMost mornings start the same way: open email, spend 20 minutes figuring out what needs a reply, draft something, realise you forgot two threads.\n\nOne Click Claw fixes that. Every morning, it scans your inbox, flags what matters, and saves the drafts. You check the summary, approve the replies. Done in 2 minutes.\n\nWe're opening free trials this week.\n\n[CTA: Try it free]\n\nILMU Claw",
      "cta_text": "Try it free",
      "cta_url": "[insert link]"
    },
    "one_pager": {
      "headline": "One Click Claw — Your inbox, handled",
      "subheadline": "AI-powered email triage that runs every morning before you start work.",
      "problem": "Inbox management eats 1–2 hours a day for most founders and operators.",
      "solution": "One Click Claw scans your inbox, summarises what matters, drafts replies in your voice, and sends you a digest — automatically.",
      "key_benefits": [
        "Daily digest delivered every morning",
        "Replies drafted, saved, ready to send — you just approve",
        "Works with Gmail today, set up in about a minute"
      ],
      "social_proof": "[insert — e.g. 'Used by 50+ SME teams across Malaysia']",
      "cta": "Start free trial at [URL]"
    }
  },
  "research_notes": "No strong direct competitors targeting Malaysian SMEs with Telegram-native delivery found. Local context angle is differentiating."
}
```

---

## Ready-to-run prompt (copy/paste into ILMU Claw)

```
Create a marketing campaign for [product/service] aimed at [target audience].

Brief:
- Tone: [e.g. "plain-spoken, confident, no buzzwords"]
- Goal: [e.g. "drive sign-ups / raise awareness / generate leads"]
- Key message: [e.g. "saves 1 hour a day on email"]
- Brand constraints: [e.g. "no em-dashes, short sentences, concrete over descriptive"]

Deliver:
1. 3 social posts — one each for LinkedIn, Twitter/X, and Telegram broadcast
2. One email newsletter (subject, preview text, body, CTA)
3. One one-pager brief (headline, problem, solution, 3 key benefits, CTA)

Do a quick web search to check if any claims need a stat to back them up.
Return all assets as JSON.
Flag if anything in the brief is vague or contradictory.
```

---

## Platform-specific formatting rules

| Platform | Max length | Format notes |
|----------|-----------|--------------|
| LinkedIn | 700 chars (organic) | Paragraph breaks every 2–3 lines. Hook in line 1. CTA last. |
| Twitter/X | 280 chars | Hook + one supporting line + CTA. Hashtags max 2. |
| Telegram broadcast | 160 chars | Conversational. One clear CTA. |
| Email newsletter | No hard limit | Subject ≤ 50 chars. Preview text ≤ 90 chars. One CTA only. |
| One-pager | One screen | Headline + 3 bullets + CTA. No paragraphs. |

---

## Execution rules for ILMU Claw

1. **Brief first.** Always confirm the brief is complete before generating. If product, audience, or goal is missing, ask before proceeding.
2. **Concrete over descriptive.** Copy must show what the product does, not describe how great it is. "Saves 1 hour on email" beats "powerful AI automation".
3. **Tone must be consistent.** All assets in one run must share the same voice. Do not mix formal and casual within a campaign.
4. **One CTA per asset.** Each piece has exactly one call to action. No lists of options.
5. **Research is optional but honest.** If web search is used to find a stat, include the source in `research_notes`. Never fabricate statistics.
6. **JSON before documents.** If a Drive document is requested, return the full JSON first, then generate the file. JSON is the source of truth.
7. **No auto-publish.** ILMU Claw does not post to social platforms, send newsletters, or publish documents. All output is draft-only.

---

## Failure handling

| Failure | Behaviour |
|---------|-----------|
| Brief is incomplete | Return a `"missing_fields"` list and ask the user to fill in before generating |
| Web search fails | Proceed without research notes; set `"research_notes": "Web search unavailable — claims unverified."` |
| Claim cannot be substantiated | Flag it in `research_notes`: "Unverified — recommend adding a source or removing." |
| Tone instructions conflict | Flag the conflict inline and ask user to clarify before generating |
| Drive save fails | Return assets as JSON in-chat; note the file save error |
