# Vault Access & Project Resolution

Throughout these skills, `[vault]` means the absolute path to the user's Obsidian vault root. Always substitute the real path when reading or writing files — never write the literal string `[vault]`.

## Resolve the vault root (in order)

Try these in order; stop at the first that succeeds.

1. **cwd CLAUDE.md** — if `./CLAUDE.md` has a `## Vault` section with an `Overview:` line, the vault root is everything before the `/work/` or `/personal/` segment of that path.
2. **User memory** — look for a line `Obsidian vault: <path>` in `~/.claude/CLAUDE.md` (written by `/vault:setup`).
3. **cwd is the vault** — if cwd or an ancestor contains a `.obsidian/` folder, that folder's parent is the vault root.
4. **Ask** — ask the user for their vault path. If the vault has no second-brain structure yet, suggest `/vault:setup`.

## Resolve the bucket and project slug (in order)

Resolves the **bucket** (`work` or `personal`) and **project slug** for the current working directory.

1. **cwd CLAUDE.md pointer** — from the same `Overview:` line, e.g.
   `Overview: [vault]/personal/projects/portfolio-site-mw/portfolio-site-mw.md`
   - **Bucket** — the `work` or `personal` path segment
   - **Project slug** — the segment after `projects/`
2. **Fallback — match by cwd folder name** — take the last folder name in cwd (e.g. `~/Developer/selective-notification` → `selective-notification`) and check `[vault]/work/projects/[name]/` then `[vault]/personal/projects/[name]/`.

Prefer the CLAUDE.md pointer over folder-name matching — it's the source of truth after `/vault:relink`.

If neither resolves, follow **No match** below.

### No match

Stop and offer the user a numbered list. Every skill's list carries these two; a skill adds its own options where it says so:

1. Run `/vault:new-project` to set one up, then re-run this skill
2. Name an existing vault project to use

`/vault:new-project` and `/vault:relink` are user-invoked: they run only when the user types the command as its own message, so offer them and wait. When the user names a project, continue with it.

The project **hub note** — its status and index file — is `[vault]/[bucket]/projects/[slug]/[slug].md`, falling back to `README.md` in older projects.

## File placement

A project folder keeps exactly three things at its root: the hub note, `sessions/`, and `docs/`. Every other file lives in a type folder under `docs/` — `specs/`, `prds/`, and the ingest taxonomy (`transcripts/`, `research/`, `articles/`, `interviews/`, `brainstorms/`, `feedback/`, `prep/`, `notes/`).

This applies to **self-authored** documents, not just ingested ones. Talking points, running question lists, requirements maps, proposals, research writeups — anything written for the project during a session files under `docs/[type]/` exactly as if it had been ingested: meeting prep and question lists → `docs/prep/`, research → `docs/research/`, anything else → `docs/notes/`. Never write a file at the project root or loose at `docs/` root, and add an index line for it under the matching hub-note section (`## Prep`, `## Research`, `## Notes`, …).

Living documents that get updated across sessions (a running question list, a requirements map) keep a plain descriptive filename with no timestamp prefix; point-in-time records keep the `YYYYMMDD-HHMM-` prefix.

## Tool access

Skills name `obsidian_*` MCP tools, from the **mcp-obsidian** bridge (Route B in `/vault:connect-obsidian`). Any Obsidian MCP is **optional**: the vault is plain Markdown on disk. But two other setups are common, and the tool names differ in each, so map before you call.

**Check which setup you are on** by looking for `vault_read` (Local REST API built-in server, Route A) or `obsidian_get_file_contents` (mcp-obsidian, Route B) among your available tools. Tools appear prefixed in-session, e.g. `mcp__obsidian__vault_read`; match on the bare name. If a skill names a tool you do not have, find its row below and use the column for your setup.

| mcp-obsidian (Route B) | REST API built-in server (Route A) | No MCP |
|---|---|---|
| `obsidian_list_files_in_vault` / `obsidian_list_files_in_dir` | `vault_list` | list the directory |
| `obsidian_get_file_contents` | `vault_read` | Read |
| `obsidian_append_content` | `vault_append` | Edit / Write |
| `obsidian_patch_content` | `vault_patch` (targeting differs, see below) | Edit / Write |
| `obsidian_simple_search` | `search_simple` | Grep |
| `obsidian_complex_search` | `search_query` (JsonLogic over tags, frontmatter, path, mtime) | Grep |
| (no equivalent) | `vault_get_document_map` (heading tree, block ids, version token) | Read |

All three routes are equivalent in effect: never skip a step because the MCP is missing, and never substitute a weaker step for one you cannot find.

One exception to the table: `vault_patch` is **not** a drop-in rename of `obsidian_patch_content`. It takes a different target syntax, and getting it wrong is silent, not an error. `vault-writes.md` gives the correct call for both.
