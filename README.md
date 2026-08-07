# vault

A Claude Code plugin that turns an Obsidian vault into an AI second brain: session logs, design specs, PRDs, ingested content, and tasks — organized per project, cross-linked, and searchable.

Works on **your** vault: run `/vault:setup` once and the skills bootstrap the structure they need. No MCP server required — skills read and write the vault as plain Markdown on disk. Optionally, `/vault:connect-obsidian` wires up an Obsidian MCP: the [Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) plugin's built-in MCP server (official, preferred) or the [mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) bridge.

## Install

```
/plugin marketplace add renamelater/vault-plugin
/plugin install vault@morriswang
```

Then, in any Claude Code session:

```
/vault:setup
```

Point it at your existing Obsidian vault (or let it create one). It scaffolds the folder structure, writes a `CLAUDE.md` navigation doc into the vault, and records the vault location so every other skill can find it.

## The system

Each project gets a folder in the vault with a **hub note** (index), `sessions/` (what happened), and `docs/` (specs, PRDs, ingested content, and working documents authored during sessions — everything files into a type folder under `docs/`, nothing loose at the project root). Code folders link to their vault project via a `## Vault` section in the code folder's `CLAUDE.md`, so skills know where to file things no matter where you run them.

## Skills

Model-invoked — these fire automatically when the situation matches (or by name):

| Skill | Fires when |
|---|---|
| `/vault:compress` | Session ending — saves a session log, reconciles action items |
| `/vault:ingest` | External content pasted (transcript, research, article, interview) — or a Notion URL shared |
| `/vault:publish` | "Publish this to Notion" — turns a vault doc into a Notion page, updates it in place on republish |
| `/vault:spec` | A design decision was settled — documents it |
| `/vault:prd` | Product requirements were defined — documents them |
| `/vault:search-sessions` | Recalling what was done or decided in a past session |
| `/vault:setup` | First-time vault initialization |
| `/vault:connect-obsidian` | Wiring up the optional Obsidian MCP (guided install of the Local REST API plugin) |

User-invoked only — these carry `disable-model-invocation`, which hides them from the model entirely. Claude never fires them on its own and won't list them among its skills. Send the command as its own message; mentioned mid-sentence it arrives as plain text, and Claude — seeing no such skill — may tell you it doesn't exist:

| Skill | Use |
|---|---|
| `/vault:new-project` | Set up a new work or personal project (code or knowledge-only) |
| `/vault:relink` | Point a code folder at an existing vault project |

## Notion

`/vault:ingest` accepts Notion URLs (single pages, page trees behind an explicit selection gate, databases as one curated file) and `/vault:publish` mirrors vault documents to Notion pages with a drift check before every overwrite. Both need a Notion MCP: on claude.ai or the desktop app enable the **Notion connector** in Settings → Connectors; on the CLI run `claude mcp add --transport http notion https://mcp.notion.com/mcp`. Auth is the MCP's own OAuth — the plugin never touches API tokens.

## Layout

- `skills/*/SKILL.md` — the 10 skills
- `skills/setup/template-vault-claude.md` — the navigation doc `/vault:setup` installs into your vault
- `references/resolve-project.md` — shared rules: vault root resolution, cwd → project matching, MCP-vs-file-tool access
- `references/vault-writes.md` — shared rules: timestamps, frontmatter quoting, hub-note index insertion

## Updating

No `version` field — every commit to this repo is a new version:

```
/plugin update vault@morriswang
```
