---
name: search-sessions
description: Search past sessions, specs, and PRDs in the Obsidian vault. Use when the user asks what was done or decided in a past session.
---

# Search Past Sessions in Obsidian

The project `CLAUDE.md` in cwd is already in context: its `## Vault` section's `Overview:` line gives the **bucket** (the `work`/`personal` segment) and **slug** (the segment after `projects/`) for a default-scoped search. No `Overview:` line, or the search is vault-wide anyway: read `${CLAUDE_PLUGIN_ROOT}/references/resolve-project.md` for the resolution ladder and the tool map for setups without Route A.

## Step 1: Determine Search Scope
- If the user provides a search term after `/vault:search-sessions`, use it
  - Example: `/vault:search-sessions notification layout` → search for "notification layout"
- If no search term is provided, default to searching sessions for the current project based on `cwd`
- If the user specifies work or personal, scope the search accordingly
- If the user is clearly looking for a decision or requirement rather than a session, include `docs/specs/` and `docs/prds/` in the scope and say so in the results

## Step 2: Search the Vault
- `search_simple` for keyword searches
- `search_query` (JsonLogic over path, tags, frontmatter, mtime) for anything more specific
- Scope search to sessions folders:
  - `work/projects/*/sessions/`
  - `personal/projects/*/sessions/`
- `vault_list` to browse one project's sessions folder
- **Ordering:** search results come back unordered. Every session filename starts with a `YYYYMMDD-HHMM` stamp — sort the matched filenames by that prefix yourself, newest first, before presenting.

## Step 3: Present Results
Show a numbered list:
```
1. [YYYY-MM-DD] [project-name] — [session title]
   Path: [vault path]

2. [YYYY-MM-DD] [project-name] — [session title]
   Path: [vault path]
```

- Show title and date only, not full content
- If no results found, suggest broader search terms

## Step 4: Offer Actions
- Ask if the user wants to see the full details of any result
- If yes, `vault_read` that session file and display it
- Offer to continue where a past session left off by reading the Open Items section
- Session files chain via the `previous:` frontmatter field — follow it to walk further back in time

## Rules
- Show title and date only unless asked for more
- Never show more than 10 results at once
- If no results found, say so clearly and suggest alternatives
