---
name: openclaw-financial-pricing-report
description: "Use this skill when the user wants ILMU Claw to fetch current prices for a stock, FX rate, commodity, or crypto asset, compare against a previous value or threshold, and deliver a formatted report. Triggers include: 'get me the price of', 'daily price alert for', 'monitor [ticker/asset]', 'send me a pricing snapshot', 'alert me if [asset] moves more than X%', 'weekly market report'. Covers both on-demand price checks and scheduled daily/weekly snapshots with threshold alerts."
---

# ILMU Claw — Financial Pricing Report

**Status:** SCHEDULABLE — on demand or daily/weekly schedule

---

## What this recipe does

1. Fetches the latest price(s) for one or more specified assets (stocks, FX, commodities, crypto)
2. Compares against the previous close or a user-defined threshold
3. Adds a one-line "why it moved" note sourced from recent news
4. Flags clearly if a threshold breach has occurred
5. Delivers a tidy summary as a message in your chat channel

---

## Tools / connectors required

| Connector | Purpose |
|-----------|---------|
| Web search | Fetch current prices + news context |

Delivery is sent back through your configured messaging channel (e.g. Telegram).

---

## Structured output schema

```json
{
  "report_date": "YYYY-MM-DD",
  "report_time": "HH:MM (timezone)",
  "assets": [
    {
      "ticker": "AAPL",
      "name": "Apple Inc.",
      "asset_type": "stock",
      "current_price": 213.45,
      "currency": "USD",
      "previous_close": 210.10,
      "change_abs": 3.35,
      "change_pct": 1.59,
      "direction": "UP",
      "threshold_set": 5.0,
      "threshold_breached": false,
      "one_line_reason": "Strong earnings guidance lifted sentiment across big tech.",
      "source_url": "https://..."
    }
  ],
  "alerts": [],
  "summary_line": "No threshold breaches today. AAPL +1.6%, USDMYR +0.3%."
}
```

### Direction values

| Value | Meaning |
|-------|---------|
| `UP` | Price increased vs previous close |
| `DOWN` | Price decreased vs previous close |
| `FLAT` | Change < 0.1% |

### Asset types supported

`stock` · `fx` · `crypto` · `commodity` · `index`

---

## Ready-to-run prompt (copy/paste into ILMU Claw)

### On-demand price check

```
Fetch the current price of [ticker/asset name].
1. Get today's price and the previous close.
2. Calculate the % change.
3. Find a one-line reason for the move from recent news.
4. Flag if the move exceeds [X]%.
Return as JSON: { "report_date", "report_time", "assets": [...], "alerts": [], "summary_line" }
Then send me the summary here in chat.
```

### Scheduled daily report

```
Every weekday at 8am, fetch prices for: [list your assets, e.g. AAPL, USD/MYR, BRENT].
For each:
1. Current price + % change from yesterday's close.
2. One-line reason for the move.
3. Flag clearly if any asset moves more than [X]%.
Return JSON, then send the formatted report in chat.
```

---

## Scheduled variant

```
Schedule: weekdays 08:00 (local time)
Assets: [your watchlist]
Threshold: 5% (default — adjust per asset if needed)
```

---

## Execution rules for ILMU Claw

1. **Always include a reason.** Never return a price change without at least one line of news context. If no news is found, state: `"one_line_reason": "No significant news found for this move."` — do not omit the field.
2. **Source the price.** Always include `source_url` pointing to where the price was fetched. Do not fabricate prices.
3. **Threshold logic is per-asset.** If different assets have different thresholds, apply them individually. Log `threshold_breached: true/false` for each.
4. **Alerts before delivery.** If any threshold is breached, lead the chat message with a 🚨 alert block before the full report.
5. **JSON before delivery.** Always emit the JSON first, then send the formatted summary.
6. **No financial advice.** The report is informational only. Do not include buy/sell recommendations or investment guidance of any kind.

---

## Failure handling

| Failure | Behaviour |
|---------|-----------|
| Price fetch fails for an asset | Set `current_price: null`, note the failure, continue with remaining assets |
| News search returns nothing | Use `"one_line_reason": "No news context found."` — do not hallucinate a reason |
| All price fetches fail | Abort, notify user in-chat: "Pricing report failed — web search may be unavailable." |

---

## Example output (sent in chat)

```
📊 Pricing Report — 23 Jun 2026 · 08:00 MYT

AAPL (Apple)       $213.45   +1.6% ↑
USD/MYR             4.71     +0.3% ↑
BRENT CRUDE        $82.10   -2.1% ↓

Reasons:
• AAPL: Strong earnings guidance lifted sentiment across big tech.
• USD/MYR: Dollar strengthened on Fed hold signal.
• Brent: IEA revised demand forecast downward.

No threshold breaches today (threshold: 5%).
```
