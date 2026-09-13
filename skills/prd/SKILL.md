---
name: prd
description: Save a PRD into the Obsidian vault. Use when the user says "prd", or product requirements have just been defined in conversation.
---

# Save PRD / Product Requirements Document

## Step 1: Resolve the Vault Project

The project `CLAUDE.md` in cwd is already in context. Its `## Vault` section's `Overview:` line is the whole resolution:

`Overview: [vault]/personal/projects/portfolio-site-mw/portfolio-site-mw.md`

- **hub note** — that path, verbatim
- **vault root** — everything before the `/work/` or `/personal/` segment
- **bucket** — that segment. **slug** — the segment after `projects/`

No `Overview:` line, or no project `CLAUDE.md` in cwd: read `${CLAUDE_PLUGIN_ROOT}/references/resolve-project.md` — it owns the resolution ladder, the tool map for setups without Route A, and no-match handling.

## Step 2: Gather — One Turn, Four Calls

Independent, so issue them together:

1. `date '+%Y%m%d-%H%M %Y-%m-%d %H:%M'` — the filename stamp and the frontmatter stamp. Never estimate either from conversation context; the model's internal clock drifts by hours.
2. `vault_get_document_map` on the hub note — the `version` token for the Step 4 patch.
3. `vault_list` `[bucket]/projects/[slug]/docs/specs/` — specs related to this PRD get linked under `## Related Specs` by filename, without `.md`.
4. `Read ${CLAUDE_PLUGIN_ROOT}/references/vault-writes.md` — the insertion procedure Step 4 follows.

If the conversation has already defined the requirements, that is the content — write it. Only when it hasn't, ask one question: "What's the feature or requirement to document?"

## Step 3: Compose the PRD

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

Any frontmatter value containing a colon, quote, bracket, or `#` goes in single quotes — titles are the usual offender.

## Step 4: Save and Index — One Turn, Two Calls

Two different files, so both writes go in the same turn:

1. `vault_append` the PRD to `[bucket]/projects/[slug]/docs/prds/YYYYMMDD-HHMM-short-title.md`
   - Example: `work/projects/selective-notification/docs/prds/20260415-1130-bulk-publish.md`
2. `vault_patch` the index line under the hub note's `## PRDs`, per the procedure in `vault-writes.md`. On Route A the `ifMatch` and `rejectIfContentPreexists` guards make the call its own confirmation — a wrong or duplicated write fails loudly rather than landing silently, so don't re-read the hub to check.

## Step 5: Confirm
- Tell the user the PRD was saved
- Show the exact file path

## Rules
- If the user provides the content, don't ask questions — just format and save it
- Omit any sections that aren't relevant
