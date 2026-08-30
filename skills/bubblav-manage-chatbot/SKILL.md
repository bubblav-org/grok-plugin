---
name: bubblav-manage-chatbot
description: |
  Read and update the BubblaV chatbot's persona, custom instructions, greeting, suggestions, bot name, and widget design (colors, position). Use this skill when the user wants to change how their chatbot sounds, what it says when it greets visitors, or how the widget looks. Triggers include "change my chatbot persona", "make the bot more formal", "update the greeting", "rebrand the widget", "change the widget color", "edit custom instructions", "update the bot name", "set up the welcome message", "position the widget on the left".
metadata:
  version: 0.1.0
---

# BubblaV — manage the chatbot

Read and update the persona, custom instructions, and widget design of the user's BubblaV chatbot. **All writes require a read-then-confirm step.** The widget is customer-visible — never overwrite a field without showing the current value and getting the user's explicit approval.

> Assumes the BubblaV MCP is already connected. If it is not, hand off to the `bubblav-mcp` hub skill first.

## When to use

- The user wants to change the **persona** or tone of the bot (formal/casual/friendly/concise).
- The user wants to update the **greeting** or welcome message.
- The user wants to change the **bot name** or **avatar**.
- The user wants to update the **suggestions** shown beneath the greeting.
- The user wants to rebrand the **widget** (primary color, text color, position, launcher icon).
- The user wants to set or edit the **custom instructions** (system prompt for the underlying model).
- The user wants to revert any of the above to a previous state.

## When NOT to use

- The user wants to change **knowledge base content** — that is a different tool family, not this skill.
- The user wants to set up **intent triggers** (human handoff rules) — those have their own tools and a different review workflow.
- The user wants to manage **forms**, **custom tools**, or **API keys** — separate skills/tools.

## Capabilities and tools

The persona/instructions and widget design live behind the following tools (names are stable; full schemas are at the docs link below):

- **Read persona** — fetch the current `custom_instructions`, bot name, tone defaults.
- **Update persona** — write a new `custom_instructions` (or merge into the existing one if the tool supports partial updates).
- **Read widget** — fetch greeting, suggestions, bot name, primary color, text color, position, launcher visibility.
- **Update widget** — write any of the above fields.

Exact tool names are listed in `tools/list` on every session — the server is the source of truth. If you are unsure of the name, call `tools/list` and look for tools whose descriptions mention persona, instructions, greeting, suggestions, or widget design.

Full reference: [https://docs.bubblav.com/developer-guide/mcp-server](https://docs.bubblav.com/developer-guide/mcp-server).

## Required workflow for any write

**Always follow this sequence. Never skip the read or the confirmation.**

1. **Read the current state.** Call the read tool for every field the user wants to change. Do not assume the field is empty or unchanged.
2. **Show the user the current value.** Quote it verbatim in the chat so the user can see what they are about to overwrite.
3. **Propose the new value.** Diff it against the current value. If the user gave freeform guidance ("make it more formal"), translate it into a concrete new value before writing.
4. **Ask for explicit confirmation.** The widget is on a live website. A confirmation like "Yes, apply" or "Looks good, go ahead" is required before any write tool is called.
5. **Call the write tool with the new value.** If the tool returns an error, surface the error verbatim — do not retry silently with a modified value.
6. **Confirm the change took effect.** Read the field back and show the new value to the user.

If the user gives a multi-field change (for example, "rebrand the widget — change the color, the bot name, and the greeting"), do all the reads up front, show a single consolidated diff, get one confirmation, then do all the writes.

## Examples

### Example 1 — update the greeting

User: "Change my chatbot's greeting to 'Hi! How can we help you today?'"

1. Read the current greeting.
2. Reply:
   > Current greeting: "Hello! 👋 How can I assist you today?"
   > Proposed: "Hi! How can we help you today?"
   > Apply?
3. Wait for confirmation.
4. Call the write tool.
5. Read it back and confirm.

### Example 2 — make the bot more formal

User: "Make my chatbot more formal."

1. Read the current `custom_instructions`.
2. Reply with the current instructions and a proposed rewrite (drop emojis, replace contractions, add a sign-off rule, tighten the closing line). Quote both side by side.
3. Wait for confirmation.
4. Write the new instructions.
5. Read back and show the new value.

### Example 3 — rebrand the widget

User: "Rebrand the widget — use the new navy color #0F2A4A, position it bottom-right, change the bot name to 'Aria'."

1. Read the current widget config (color, position, name).
2. Reply with a single diff:
   > bot_name:        "Helper Bot"  →  "Aria"
   > primary_color:   "#3B82F6"     →  "#0F2A4A"
   > position:        "bottom-left" →  "bottom-right"
   > Apply all three?
3. Wait for confirmation.
4. Call the update tool with the three fields.
5. Read back and confirm.

## Anti-patterns

- **Do not write without reading first.** You do not know what is there.
- **Do not write without confirmation.** The widget is on a live site.
- **Do not call `update_*` with an empty object** to "clear" a field unless the user explicitly asked to clear it.
- **Do not translate the user's casual guidance ("a bit warmer") into a concrete new value silently.** Show the translation and get confirmation.
- **Do not batch unrelated changes** ("and also update the API key") into the same confirmation — split them so the user can approve one at a time.
- **Do not invent field names.** If a tool's schema does not accept a field the user asked to change, say so and offer the closest supported alternative.
