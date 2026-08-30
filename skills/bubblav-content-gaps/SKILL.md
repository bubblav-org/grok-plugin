---
name: bubblav-content-gaps
description: |
  Discover what questions visitors are asking that the BubblaV chatbot cannot answer, and recommend specific sources to add. Use this skill when the user wants to find knowledge gaps, see unanswered questions, audit their bot's coverage, or decide what new content to write. Triggers include "what questions am I missing", "knowledge gaps", "unanswered questions", "what can't my bot answer", "content audit", "find gaps in my knowledge base", "what should I add to my docs".
metadata:
  version: 0.1.0
---

# BubblaV — find content gaps

Surface the questions real visitors are asking that the user's BubblaV chatbot could not answer, and recommend specific sources to close the gaps.

> Assumes the BubblaV MCP is already connected. If it is not, hand off to the `bubblav-mcp` hub skill first.

## When to use

- The user wants to know what their bot is failing at and why.
- The user is planning a content sprint and wants a prioritized list of topics to write or scrape.
- The user is auditing coverage of a product area ("are we missing anything about billing?").
- The user is doing a periodic review (weekly, monthly) of support quality.

## When NOT to use

- The user wants the bot to answer a specific question right now — use `bubblav-search-knowledge`.
- The user wants to add a single known page to the knowledge base — that is a direct management tool call, not this skill.

## Primary tool

### `mcp__bubblav__bubblav_list_unanswered_questions`

Returns recent visitor questions that the bot could not answer (low confidence, no matching knowledge entry, or escalated to a human).

**Parameters**

| Name | Type | Required | Default | Notes |
|------|------|----------|---------|-------|
| `date_range` | object | no | last 7 days | `{ start: "yyyy-MM-dd", end: "yyyy-MM-dd" }` |
| `limit` | number | no | 50 | 1–200 |
| `min_count` | number | no | 1 | Only return questions asked at least N times — filters out one-offs |

**Return shape (curated)**

```json
{
  "questions": [
    {
      "question": "Do you offer student discounts?",
      "count": 14,
      "last_asked": "2026-08-28T15:22:00Z",
      "sample_conversation_ids": ["..."]
    }
  ],
  "total": 87
}
```

## Workflow

1. **Pick a sensible window.** Default to the last 7 days. Expand to 30 days for a monthly review. Avoid going back further than 90 days — older questions usually reflect outdated product surfaces.
2. **Raise `min_count` to filter noise.** A good first pass is `min_count: 2` (questions asked more than once). For a deep audit, drop it to `1` and let the volume speak.
3. **Group by topic.** Cluster the returned questions into 3–7 themes. Common groupings: pricing/billing, account/login, integration/setup, product feature X, shipping/returns, etc. The model does the clustering; do not require the tool to return it.
4. **For each cluster, recommend a concrete action:**
   - **Add a public source.** If a relevant page already exists on the user's site but was not indexed, recommend `bubblav_scrape_url` against that URL (pair with `bubblav-search-knowledge` to verify whether it is already in the index).
   - **Write new content.** If no page covers the topic, recommend the user create one and then add it.
   - **Re-route to human.** If the topic is intentionally not in the bot's scope (for example, legal advice), recommend an intent trigger that hands off to a live agent instead.
5. **Prioritize by `count`.** Sort clusters by total question volume so the user fixes the highest-impact gaps first.
6. **Do not invent gaps.** Every recommendation must be grounded in a returned question, not in the model's general knowledge of the user's product.

## Examples

### Example 1 — weekly content gap review

User: "What questions am I missing this week?"

```
mcp__bubblav__bubblav_list_unanswered_questions({
  date_range: { start: "<7 days ago>", end: "<today>" },
  min_count: 2
})
```

Then respond with a 3–7-bucket summary, each bucket showing the top questions and a one-line "what to do" recommendation. End with a single prioritized list of the top 5 questions by `count` and the proposed source for each.

### Example 2 — auditing a single product area

User: "Are we missing anything about billing?"

Filter the returned questions to anything containing "bill", "invoice", "charge", "refund", "subscription", "plan", "price", "payment". Cluster the matches. Recommend specific source pages or new content for each cluster.

### Example 3 — single-question deep dive

User: "Why is my bot failing on 'Do you ship to Norway?'?"

```
mcp__bubblav__bubblav_list_unanswered_questions({
  date_range: { start: "<30 days ago>", end: "<today>" }
})
```

Filter to the specific question. Cross-reference with `mcp__bubblav__bubblav_search_knowledge` to confirm the topic is missing from the index. Recommend either adding a shipping policy page or extending the existing one to cover Norway specifically.

## Anti-patterns

- **Do not invent questions.** Only report what the tool returns.
- **Do not collapse distinct questions into one cluster just because they share a word.** A question about "refund policy" and a question about "refund processing time" may need different sources.
- **Do not recommend scraping an internal page or a competitor's site.** The knowledge base is for the user's own content.
- **Do not run this skill more than once per session unless the user asks.** It is expensive on the server side and the results do not change minute-to-minute.
