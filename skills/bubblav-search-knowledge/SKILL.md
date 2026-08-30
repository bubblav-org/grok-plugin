---
name: bubblav-search-knowledge
description: |
  Semantic search over the user's BubblaV-indexed website content with citations. Use this skill whenever the user asks to search their knowledge base, find articles on their site, look up what their help center says about a topic, or wants a sourced answer from their own content. Triggers include "search my knowledge base", "what does my site say about", "find an article on", "look up in my docs", "check my help center for", "what's in my knowledge base about", "search bubblav for".
metadata:
  version: 0.1.0
---

# BubblaV — search knowledge base

Semantic search over the website content the user has trained their BubblaV chatbot on. The single tool for this skill is `mcp__bubblav__bubblav_search_knowledge`.

> Assumes the BubblaV MCP is already connected. If it is not, hand off to the `bubblav-mcp` hub skill first.

## When to use

- The user wants a sourced answer from their own content (not from the model's training data).
- The user is debugging what their bot would say in response to a question.
- The user wants to verify that a specific page, policy, or fact is in the bot's knowledge base.
- The user is preparing a customer-support response and needs the canonical reference.

## When NOT to use

- The user wants to **add** new content to the knowledge base — that is a separate management tool, not search. Tell the user which tool to use or hand off to `bubblav-manage-chatbot` for the broader workflow.
- The user wants to find **gaps** in the knowledge base — use `bubblav-content-gaps`.
- The user wants general web search — do not use this skill; the knowledge base only contains the user's own indexed content.

## Tool

### `mcp__bubblav__bubblav_search_knowledge`

**Parameters**

| Name | Type | Required | Default | Notes |
|------|------|----------|---------|-------|
| `query` | string | yes | — | Natural-language question or phrase. Be specific. |
| `limit` | number | no | 5 | 1–20. Raise to 10–20 for exhaustive research; keep at 5 for a single sourced answer. |

**Return shape**

```json
{
  "results": [
    {
      "content": "The content snippet...",
      "source": "https://example.com/page",
      "title": "Page Title",
      "relevance": 0.95
    }
  ],
  "total": 42
}
```

## Workflow

1. **Read the query literally.** Do not paraphrase or expand it before searching — the embedding model is already tolerant of natural phrasing.
2. **Pick a `limit`.** Default to `5`. Raise to `10`–`20` only when the user wants exhaustive coverage ("every article that mentions…", "all pages about…").
3. **Call the tool once with the best query.** Avoid chaining many narrow searches when one broader query with a higher `limit` will do — it costs the user API credits and clutters the context window.
4. **Synthesize with citations.** When you answer the user, link back to the `source` URL of every result you relied on. Do not paraphrase a result's `content` into something that changes its meaning.
5. **Handle empty results honestly.** If `results` is empty or `total` is 0, say so. Suggest the user add the missing content via the knowledge-base management tools (or hand off to `bubblav-content-gaps` to discover what is missing systematically).

## Examples

### Example 1 — single sourced answer

User: "What does my help center say about shipping to Germany?"

```
mcp__bubblav__bubblav_search_knowledge({ query: "shipping to Germany" })
```

Reply (model output, not a tool):

> According to your help center ([Shipping policy](https://example.com/shipping)), orders to Germany arrive in 5–7 business days via DHL. Duties are paid by the recipient for orders under €150.

### Example 2 — exhaustive research

User: "List every page on my site that mentions GDPR."

```
mcp__bubblav__bubblav_search_knowledge({ query: "GDPR", limit: 20 })
```

Then enumerate the `source` URLs and a one-line summary of each, grouped by topic (legal pages, blog posts, product pages, etc.).

### Example 3 — empty result

If the call returns no results:

> I could not find anything in your knowledge base about that topic. A few likely reasons: the page has not been crawled yet, it is excluded by your sitemap config, or the topic is not actually covered on your site. Want me to (a) add this URL to the crawl queue, (b) paste the source text directly into a new knowledge entry, or (c) generate a list of related questions your bot is being asked but cannot answer (the `bubblav-content-gaps` skill)?

## Anti-patterns

- **Do not invent sources.** If a result is not in the tool output, do not cite it. The whole point of this skill is ground-truth.
- **Do not loop searches.** If a single search with a higher `limit` would answer the question, do not call the tool multiple times with reformulated queries.
- **Do not include `relevance` in the user-facing answer.** It is an internal ranking signal, not a confidence statement to the user.
- **Do not skip the citation.** Every claim drawn from a search result must link back to its `source` URL.
