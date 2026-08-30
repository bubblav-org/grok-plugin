# BubblaV for Grok Build

BubblaV is an AI chatbot platform that trains on your website content and answers customer questions 24/7. This plugin wires the official **BubblaV remote MCP server** into Grok Build, so you can search your knowledge base, tune your chatbot persona and widget, analyze conversations, and close content gaps — without leaving the chat.

> Tagline: _Deploy an AI chatbot in minutes. Answer customer questions 24/7. No coding required._

## Install

In Grok Build, run `/plugin`, search for **bubblav**, and install. The install wires up the MCP automatically — no global npm install, no `npx`, no API key to copy.

## Authenticate

The first time Grok uses a BubblaV tool, you will be prompted to sign in through your browser (OAuth). Approve it once and you are connected. You can check the connection at any time with `/mcp`.

### Headless / API-key clients

If you need a long-lived credential (CI, scheduled agents, headless bots), generate a scoped MCP API key in your BubblaV dashboard and call the API-style endpoint:

1. Sign in at [https://www.bubblav.com](https://www.bubblav.com)
2. Open **Website Settings → API Keys → Generate New Key**
3. Choose a name and select one or both scopes:
   - `mcp:read` — read-only access to your website data
   - `mcp:tools:execute` — execute MCP tools and scrape URLs
4. Copy the key — it is shown **once**

Then point your client at the API-style endpoint with the key in the `X-API-Key` header:

```
URL:    https://www.bubblav.com/api/mcp
Header: X-API-Key: bubblav_mcp_...
```

Full setup details: [https://docs.bubblav.com/developer-guide/mcp-server](https://docs.bubblav.com/developer-guide/mcp-server)

## Skills

This plugin bundles five skills. They are auto-loaded when the conversation matches their trigger phrases.

| Skill | Use it for |
|-------|------------|
| `bubblav-mcp` | Hub skill — connect BubblaV, list available tools, manage API keys, scope selection |
| `bubblav-search-knowledge` | Find answers in your indexed website content with citations |
| `bubblav-content-gaps` | Surface unanswered questions and recommend new sources |
| `bubblav-manage-chatbot` | Update persona, greeting, custom instructions, widget design |
| `bubblav-analyze-reports` | Summarize containment, peak hours, ratings, leads, and trends |

## MCP tools

The connected client receives the **complete, always-current** tool list via the standard `tools/list` method. The five most-used tools, with current docs, are:

- `bubblav_search_knowledge` — semantic search over your indexed content
- `bubblav_read_report` — full analytics report (current and previous period)
- `bubblav_list_unanswered_questions` — questions your bot could not answer
- `bubblav_scrape_url` — convert any public URL into markdown
- `bubblav_update_chatbot` — update persona, instructions, and widget design

For the full reference (40+ tools covering knowledge base, analytics, conversations, leads, forms, API keys, custom tools, and intent triggers), see the [MCP server documentation](https://docs.bubblav.com/developer-guide/mcp-server).

## Usage examples

Once installed, just ask naturally:

```
What does my help center say about shipping to Germany?
What questions did my chatbot fail to answer last week?
Make the bot more formal and update the greeting to "Hi! How can I help?"
Summarize last month's chatbot performance and call out anything unusual.
What were the peak support hours this week?
```

## Security

This plugin declares exactly one network endpoint: `https://www.bubblav.com/mcp` (and its API-style alias `https://www.bubblav.com/api/mcp` for headless use). All other requests originate from your installed client, not from the plugin.

- **No `postinstall` scripts.** No `curl | bash`. No shell hooks. No lifecycle hooks of any kind.
- **No telemetry** is sent from the plugin. The BubblaV server records an audit log of MCP tool calls (tool name, arguments, timestamp, caller IP) for security and billing — visible to you in **Dashboard → MCP Logs**.
- **OAuth** is the default auth path. API keys are scoped (`mcp:read` and/or `mcp:tools:execute`) and revocable from the dashboard at any time.
- **No bundled binaries.** The plugin is JSON + Markdown. There is nothing to execute locally.

If you find a security issue, email [security@bubblav.com](mailto:security@bubblav.com).

## Resources

- **Product**: [https://bubblav.com](https://bubblav.com)
- **Documentation**: [https://docs.bubblav.com](https://docs.bubblav.com)
- **MCP setup guide**: [https://docs.bubblav.com/developer-guide/mcp-server](https://docs.bubblav.com/developer-guide/mcp-server)
- **Plugin source**: [https://github.com/bubblav-org/grok-plugin](https://github.com/bubblav-org/grok-plugin)

## License

MIT — see [LICENSE](./LICENSE).
