---
name: bubblav-analyze-reports
description: |
  Read and summarize the BubblaV chatbot analytics report — total conversations, containment rate, average response time, message rating, lead capture, hourly activity, top countries, and period-over-period comparisons. Use this skill when the user wants a performance summary, a weekly/monthly review, peak-hours analysis, or a health check on their bot. Triggers include "summarize last month", "how is my chatbot doing", "containment rate", "average response time", "peak support hours", "how many leads did we capture", "what's my bot's rating", "weekly review", "monthly review", "analyze my bubblav reports".
metadata:
  version: 0.1.0
---

# BubblaV — analyze reports

Read the user's BubblaV analytics and turn it into a short, prioritized summary. Lead with one headline number, then 2–3 supporting metrics, then 1–2 actionable observations.

> Assumes the BubblaV MCP is already connected. If it is not, hand off to the `bubblav-mcp` hub skill first.

## When to use

- The user wants a performance summary for a specific period.
- The user wants to know when to staff live agents (peak hours).
- The user is preparing a stakeholder update or weekly review.
- The user wants to track trends over time (period-over-period).
- The user is investigating a regression ("ratings dropped last week").

## When NOT to use

- The user wants to find **unanswered questions** or **knowledge gaps** — use `bubblav-content-gaps`.
- The user wants to inspect a **single conversation** — that is a different tool, not this skill.
- The user wants to export raw data — point them to the dashboard or the CSV export tool, do not paste hundreds of rows into the chat.

## Primary tool

### `mcp__bubblav__bubblav_read_report`

Returns the full analytics report for the user's selected website — the same data shown on the Reports page in the dashboard.

**Parameters**

| Name | Type | Required | Default | Notes |
|------|------|----------|---------|-------|
| `date_range` | object | no | current calendar month | `{ start: "yyyy-MM-dd", end: "yyyy-MM-dd" }` |

**Return shape (curated)**

```json
{
  "totalConversations": 1234,
  "totalMessages": 4567,
  "containmentRate": 0.69,
  "agentTransferRate": 0.31,
  "avgResponseTime": 12.5,
  "avgConversationDepth": 4.2,
  "messageRating": 4.1,
  "avgConfidenceScore": 0.85,
  "estimatedCostSavings": 1234.56,
  "totalLeads": 42,
  "topCountries": ["US", "GB", "DE"],
  "mostVisitedLinks": ["https://example.com/pricing"],
  "previousPeriodComparison": { /* same shape, previous period */ }
}
```

The full schema (including hourly activity, top topics, and per-channel breakdowns) is at [https://docs.bubblav.com/developer-guide/mcp-server](https://docs.bubblav.com/developer-guide/mcp-server).

## Workflow

1. **Pick the right period.** If the user did not specify, default to the current calendar month. For "last week", use the previous Monday–Sunday. For "last month", use the previous calendar month. Always pass an explicit `date_range` — do not rely on the default when the period is ambiguous.
2. **Call the tool once.** Do not call it again for "more detail" unless the user asks — the return shape already covers the common needs.
3. **Lead with a single headline number.** Choose the number that best answers the user's actual question:
   - "How is the bot doing?" → containment rate (e.g. "69% of conversations were fully handled by the bot")
   - "How many leads?" → `totalLeads`
   - "Are customers happy?" → `messageRating` (out of 5)
4. **Follow with 2–3 supporting metrics** that are directly relevant to the headline. Drop the rest — do not dump the full shape into the chat.
5. **Add one period-over-period callout** when the change is meaningful. The threshold for "meaningful" is roughly:
   - containment rate: ±3 percentage points
   - message rating: ±0.2
   - response time: ±2 seconds
   - total conversations: ±20%
6. **Close with 1–2 actionable observations** drawn from the data:
   - "Peak hour is 10:00 — consider staffing live agents then."
   - "Rating dropped 0.3 vs. last month — pair this with `bubblav-content-gaps` to find the cause."
   - "Containment is up 5 points since you added the billing docs last week."

If the user asked a specific question ("what was peak support hours?"), do not pad the response with every other metric — answer the question and stop.

## Examples

### Example 1 — monthly review

User: "Summarize last month."

1. Call `bubblav_read_report` with the previous calendar month.
2. Reply:

> **Last month: 1,234 conversations, 69% contained by the bot.**
>
> - Containment: 69% (up 4 pts vs. previous month)
> - Avg response time: 12.5s
> - Message rating: 4.1 / 5
> - Leads captured: 42
>
> Peak hour: 10:00. Top countries: US, GB, DE.
>
> **Actions to consider:** ratings on the "refund" topic are pulling the average down — run `bubblav-content-gaps` filtered to refunds to find the missing source.

### Example 2 — peak hours

User: "What are peak support hours this week?"

1. Call `bubblav_read_report` with the current week.
2. Use the hourly activity breakdown in the response.
3. Reply with a 3-row table: hour, conversations, % of daily total. Highlight the top 1–2 hours.

### Example 3 — investigating a regression

User: "Ratings dropped last week — what happened?"

1. Call `bubblav_read_report` for last week and the week before.
2. Diff `messageRating`, `avgConfidenceScore`, and the per-topic breakdown.
3. Reply with: rating delta, the topics driving the drop, and a one-line recommendation (usually "run `bubblav-content-gaps` on the worst-performing topic").

## Anti-patterns

- **Do not dump the full return shape.** Pick the 3–5 numbers that answer the user's question. The rest is noise.
- **Do not invent metrics.** Only report what the tool returns.
- **Do not round percentages inconsistently.** Round containment to whole percent; round response time to one decimal; round rating to one decimal. Same units throughout.
- **Do not compare non-comparable periods.** "Last 7 days" vs. "previous 7 days" is comparable; "this month so far" vs. "last full month" is not.
- **Do not give generic advice.** Every action item must reference a specific metric, topic, or hour from this report.
- **Do not export the raw JSON.** If the user wants the underlying data, point them to the dashboard or the CSV export.
