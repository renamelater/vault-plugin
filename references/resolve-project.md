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

If neither resolves, return to the calling skill's no-match handling.

The project **hub note** — its status and index file — is `[vault]/[bucket]/projects/[slug]/[slug].md`, falling back to `README.md` in older projects.

## File access

Skills name `obsidian_*` MCP tools (from mcp-obsidian). Any Obsidian MCP is **optional** — the vault is plain Markdown on disk. When a named tool is unavailable, use the equivalent tool from the Local REST API plugin's built-in MCP server (if connected), or plain file tools on `[vault]/...` paths:

| MCP tool | Fallback |
|---|---|
| `obsidian_list_files_in_vault` / `obsidian_list_files_in_dir` | list the directory |
| `obsidian_get_file_contents` | Read |
| `obsidian_append_content` / `obsidian_patch_content` | Edit / Write |
| `obsidian_simple_search` / `obsidian_complex_search` | Grep |

Both routes are equivalent; never skip a step because the MCP is missing.
