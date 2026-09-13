---
name: spec
description: Save a design decision into the Obsidian vault as a spec. Use when the user says "spec", or a design decision has just been settled in conversation.
---

# Save Spec / Design Decision

## Step 1: Resolve the Vault Project

The project `CLAUDE.md` in cwd is already in context. Its `## Vault` section's `Overview:` line is the whole resolution:

`Overview: [vault]/personal/projects/portfolio-site-mw/portfolio-site-mw.md`

- **hub note** — that path, verbatim
- **vault root** — everything before the `/work/` or `/personal/` segment
- **bucket** — that segment. **slug** — the segment after `projects/`

No `Overview:` line, or no project `CLAUDE.md` in cwd: read `${CLAUDE_PLUGIN_ROOT}/references/resolve-project.md` — it owns the resolution ladder, the tool map for setups without Route A, and no-match handling.

## Step 2: Gather — One Turn, Three Calls

Independent, so issue them together:

1. `date '+%Y%m%d-%H%M %Y-%m-%d %H:%M'` — the filename stamp and the frontmatter stamp. Never estimate either from conversation context; the model's internal clock drifts by hours.
2. `vault_get_document_map` on the hub note — the `version` token for the Step 4 patch, and it shows whether the section is `## Specs` or an older project's `## Decisions`.
3. `Read ${CLAUDE_PLUGIN_ROOT}/references/vault-writes.md` — the insertion procedure Step 4 follows.

If the conversation has already settled the decision, that is the content — write it. Only when it hasn't, ask one question: "What's the decision or spec to document?"

## Step 3: Compose the Spec

```
---
type: spec
project: <project-name>
date: <YYYY-MM-DD HH:mm>
tags:
  - spec
  - <project-name>
---

# <Brief title describing the decision or spec>

## What
<What was decided, designed, or specced>

## Why
<Rationale — why this approach over alternatives>

## Details
<Specifics — states, logic, edge cases, measurements, interactions, component notes>

## Figma
<Figma link if relevant, otherwise omit>

## Impact
<What this affects — components, flows, other decisions>

## Open Questions
<Anything still unresolved>
```

Any frontmatter value containing a colon, quote, bracket, or `#` goes in single quotes — titles are the usual offender.

Don't link today's session log: at spec time it usually doesn't exist yet. `/vault:compress` links session → spec at the end of the session, and that single direction is the source of truth.

## Step 4: Save and Index — One Turn, Two Calls

Two different files, so both writes go in the same turn:

1. `vault_append` the spec to `[bucket]/projects/[slug]/docs/specs/YYYYMMDD-HHMM-short-title.md`
   - Example: `work/projects/selective-notification/docs/specs/20260413-1415-notification-states.md`
2. `vault_patch` the index line under the hub note's `## Specs`, per the procedure in `vault-writes.md`. On Route A the `ifMatch` and `rejectIfContentPreexists` guards make the call its own confirmation — a wrong or duplicated write fails loudly rather than landing silently, so don't re-read the hub to check. If the hub still uses an older project's `## Decisions`, index there and rename the heading to `## Specs` while you're in the file.

## Step 5: Confirm
- Tell the user the spec was saved
- Show the exact file path

## Rules
- If the user provides the content, don't ask questions — just format and save it
- Omit any sections that aren't relevant to the specific decision
