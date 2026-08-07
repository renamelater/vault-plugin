# Notion Access

Shared reference for skills that read or write Notion (`ingest`, `publish`). All Notion access flows through a Notion MCP and its OAuth — the plugin never handles API tokens; if a token is offered, decline it and use the connect branch below.

## Find the tools

Notion MCP tools carry `notion` in their names. Servers differ in exact naming (the claude.ai connector, Notion's hosted MCP), so match by purpose:

| Purpose | Typical tool |
|---|---|
| Fetch a page or database by URL/ID | `notion-fetch` |
| Search the workspace | `notion-search` |
| Create a page | `notion-create-pages` |
| Update a page | `notion-update-page` |

If the environment lists tools as deferred (name only, no schema), load them with ToolSearch before calling.

## Notion URLs

Treat as Notion: links on `notion.so`, `*.notion.site`, `app.notion.com` — and bare page IDs (32 hex chars, dashed or not), which `fetch` accepts directly.

## No Notion tools: the connect branch

When the session has no `notion` tools, pause the skill and tell the user how to connect, then continue once the tools appear:

- **claude.ai / desktop app** — Settings → Connectors → enable **Notion**, approve the OAuth prompt
- **Claude Code CLI** — run:
  ```bash
  claude mcp add --transport http notion https://mcp.notion.com/mcp
  ```
  then authenticate via `/mcp`

A fresh session may be needed for the tools to appear; say so.

## The drift stamp

Connector MCPs do not expose `last_edited_time` on fetched pages, so drift detection runs on a **content hash**: right after any publish, fetch the page once, save the returned page content (the `<content>` block, verbatim) to a scratch file, and hash it — `shasum -a 256`, first 16 characters. Fetch output is byte-stable for an unchanged page and refreshes on edit, so a matching hash means an untouched page and a mismatch means someone edited it. Compare a hash only against a hash of the same page's fetched content.
