---
name: bubblav-mcp
description: |
  Connect to the BubblaV remote MCP server, list available tools, generate scoped API keys, and decide which companion skill to use. Use this skill whenever the user mentions BubblaV in a setup context, asks to "connect bubblav", "set up bubblav", "configure bubblav mcp", "what tools does bubblav expose", "generate a bubblav api key", "bubblav scopes", or asks how to talk to their BubblaV account from Grok.
metadata:
  version: 0.1.0
---

# BubblaV MCP — hub skill

This is the entry point for the BubblaV plugin. The plugin registers a single MCP server named **`bubblav`**. When the server is connected, the agent has access to every `mcp__bubblav__*` tool that the server currently exposes.

> This skill is the **only** skill in the plugin that handles setup and credentialing. The other four skills (`bubblav-search-knowledge`, `bubblav-content-gaps`, `bubblav-manage-chatbot`, `bubblav-analyze-reports`) assume the connection already exists and only call individual tools.

## When to use

Use this skill when the user wants to:

- connect or reconnect their BubblaV account
- see which tools BubblaV exposes in this session
- generate, list, or revoke an MCP API key
- understand the difference between `mcp:read` and `mcp:tools:execute`
- route a request to the right companion skill

## When NOT to use

- Searching indexed content → `bubblav-search-knowledge`
- Finding knowledge gaps → `bubblav-content-gaps`
- Editing persona, instructions, or widget design → `bubblav-manage-chatbot`
- Reading analytics → `bubblav-analyze-reports`

## Authentication

### Browser OAuth (default for Grok Build, Claude, ChatGPT, Antigravity)

The first time any `mcp__bubblav__*` tool is called, the client opens a browser window to `https://www.bubblav.com`. The user signs in, picks a website, and approves the requested scopes. The token is stored by the client and refreshed transparently.

No API key to copy. No `.env` to edit. No global install.

### API key (headless / CI / scheduled agents)

1. Open [https://www.bubblav.com](https://www.bubblav.com) → **Website Settings → API Keys → Generate New Key**
2. Give the key a name (for example, `"grok-ci"`).
3. Select one or both scopes:
   - `mcp:read` — read-only access to website data (analytics, knowledge base, conversations)
   - `mcp:tools:execute` — execute tools that mutate state (update persona, send messages, scrape URLs)
4. Copy the key. **It is shown once.**
5. Point the client at `https://www.bubblav.com/api/mcp` with header `X-API-Key: bubblav_mcp_...`.

The full setup guide, including OAuth callbacks and troubleshooting, lives at [https://docs.bubblav.com/developer-guide/mcp-server](https://docs.bubblav.com/developer-guide/mcp-server).

## Tool discovery

The connected MCP server advertises its full tool list on every session via the standard `tools/list` method. **Do not hardcode a tool list in this skill** — BubblaV ships new tools regularly, and the live list is always authoritative.

If the user asks "what can BubblaV do?", list the `mcp__bubblav__*` tools currently in the session, grouped by domain:

- **Knowledge base** — search, list, add, delete entries; sync tickets; manage crawls and sitemaps
- **Analytics** — read report, hourly activity, top countries, most-visited links
- **Conversations & leads** — list, search, inspect conversations and captured leads
- **Content gaps** — list unanswered questions
- **Widget design** — bot name, greeting, suggestions, colors, position
- **Persona** — `custom_instructions`
- **Scraping** — convert a public URL to markdown
- **Custom tools** — create, update, delete, toggle webhook tools (Pro+)
- **Forms** — create, update, delete AI forms
- **API keys** — list, create, revoke
- **Intent triggers / human handoff** — create and manage routing rules (Pro+)
- **Automations** — build workflows over BubblaV data

The full reference (parameters, return shapes, examples) is at [https://docs.bubblav.com/developer-guide/mcp-server](https://docs.bubblav.com/developer-guide/mcp-server).

## Routing the user to the right companion skill

When the user's request matches a specific capability, hand off to the matching skill:

| User intent | Companion skill |
|-------------|-----------------|
| "What does my site say about…", "find an article on…", "search my knowledge base" | `bubblav-search-knowledge` |
| "What questions am I missing?", "unanswered questions", "knowledge gaps" | `bubblav-content-gaps` |
| "Change my chatbot persona", "update the greeting", "rebrand the widget" | `bubblav-manage-chatbot` |
| "Summarize last month", "containment rate", "peak hours", "how is the bot doing" | `bubblav-analyze-reports` |

## Security notes

- Always show the user the **current** value before overwriting any persona, instruction, or widget field. See `bubblav-manage-chatbot` for the read-then-confirm workflow.
- If a request would require a scope the user has not granted, stop and ask. Do not try to escalate or work around scope failures.
- API keys are shown **once** at creation time. If the user has lost one, revoke it and issue a new one — there is no "reveal" path.
- Audit logs of every MCP call (tool, redacted args, timestamp, caller) are visible to the user at **Dashboard → MCP Logs**. Tell the user about this when they first connect.
