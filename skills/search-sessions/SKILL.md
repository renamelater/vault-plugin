---
name: search-sessions
description: Search past sessions, specs, and PRDs in the Obsidian vault. Use when the user asks what was done or decided in a past session.
---

# Search Past Sessions in Obsidian

When invoked, do the following:

Read `../../references/resolve-project.md` (relative to this skill's folder) before touching the vault: it owns the vault root and `[vault]` substitution, project resolution (**bucket** and **slug**), file placement, tool access, and no-match handling.

## Step 1: Determine Search Scope
- If the user provides a search term after `/vault:search-sessions`, use it
  - Example: `/vault:search-sessions notification layout` → search for "notification layout"
- If no search term is provided, default to searching sessions for the current project based on `cwd`
- If the user specifies work or personal, scope the search accordingly
- If the user is clearly looking for a decision or requirement rather than a session, include `docs/specs/` and `docs/prds/` in the scope and say so in the results

## Step 2: Search the Vault
- Use `obsidian_simple_search` for basic keyword searches
- Use `obsidian_complex_search` for more specific queries
- Scope search to sessions folders:
  - `work/projects/*/sessions/`
  - `personal/projects/*/sessions/`
- Or use `obsidian_list_files_in_dir` to browse a specific project's sessions folder
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
- If yes, use `obsidian_get_file_contents` to read and display that session file
- Offer to continue where a past session left off by reading the Open Items section
- Session files chain via the `previous:` frontmatter field — follow it to walk further back in time

## Rules
- Keep results concise — title and date only unless asked for more
- Never show more than 10 results at once
- If no results found, say so clearly and suggest alternatives
