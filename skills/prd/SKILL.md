---
name: prd
description: Save a PRD into the Obsidian vault. Use when the user says "prd", or product requirements have just been defined in conversation.
---

# Save PRD / Product Requirements Document

When invoked, do the following:

## Step 1: Resolve the Vault Project

Read `../../references/resolve-project.md` (relative to this skill's folder) before touching the vault: it owns the vault root and `[vault]` substitution, project resolution (**bucket** and **slug**), file placement, tool access, and no-match handling. Resolve the bucket and slug with it now. If they resolve, continue to Step 2. No match: follow its no-match handling.

## Step 2: Find the Hub Note and Related Specs
- Locate the project's **hub note** (defined in the shared reference from Step 1)
- Check `[project-path]/docs/specs/` for any specs related to this PRD
- Note the filenames (without `.md`) — they will be linked in the PRD

## Step 3: Ask What to Document
If the user hasn't already provided the content, ask:
- "What's the feature or requirement to document?"

Keep it to one question. If context is already clear from the conversation, skip asking and use that context directly.

## Step 4: Create the PRD Entry
Write a markdown file with this structure:

```
---
type: prd
project: <project-name>
date: <YYYY-MM-DD HH:mm>
tags:
  - prd
  - <project-name>
---

# <Feature or requirement title>

## Problem
<What problem does this solve and for whom>

## Goals
<What success looks like — be specific>

## Non-goals
<What is explicitly out of scope>

## Requirements
<Functional requirements — what the feature must do>

## Success Metrics
<How we'll know this is working>

## Related Specs
<Only include if related specs exist>
- [[spec-filename]]

## Open Questions
<Anything still unresolved>
```

## Step 5: Save to Vault

Before constructing the filename or the `date:` field, read `../../references/vault-writes.md` and follow its Timestamps and Frontmatter rules — it also owns the hub-note insertion procedure used in Step 6.

- Use `obsidian_append_content` to save the file
- Target path: `[bucket]/projects/[project-slug]/docs/prds/YYYYMMDD-HHMM-short-title.md`
  - Example: `work/projects/selective-notification/docs/prds/20260415-1130-bulk-publish.md`

## Step 6: Update the Hub Note

- Add an **index line** for the new PRD under `## PRDs`, following the insertion procedure in `vault-writes.md` exactly.

## Step 7: Confirm
- Tell the user the PRD was saved
- Show the exact file path

## Rules
- Be concise — capture enough to understand the requirements later without over-documenting
- If the user provides the content, don't ask questions — just format and save it
- Omit any sections that aren't relevant
