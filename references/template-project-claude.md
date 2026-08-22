# Project CLAUDE.md Template

The `CLAUDE.md` that `/vault:new-project` and `/vault:relink` write into a code folder. Both skills copy this block verbatim, substituting the placeholders; this file is the only place its content lives, so a change here reaches both.

Placeholders:

- `[Project Name]`: the hub note's H1, or the slug if it has none
- `[vault]`: the vault root (see `resolve-project.md`)
- `[bucket]`: `work` or `personal`
- `[slug]`: the vault project folder name
- `[hub-note-filename]`: `[slug].md`; `README.md` only for older projects that still use it
- `[one line description]`: the hub note's `## What is this` line
- `[cwd]`: the absolute path of the code folder

```
# [Project Name]

## Context
Full project context lives in the Obsidian vault:
`[vault]/[bucket]/projects/[slug]/`

Always read the vault hub note before starting work.
Always read the most recent file in the vault sessions/ folder to see where we left off.

## About this project
[one line description]

## Vault
- Overview: [vault]/[bucket]/projects/[slug]/[hub-note-filename]
- Sessions: [vault]/[bucket]/projects/[slug]/sessions/
- Specs: [vault]/[bucket]/projects/[slug]/docs/specs/
- PRDs: [vault]/[bucket]/projects/[slug]/docs/prds/

## Code folder
[cwd]

## Rules
- At the end of every session, run /vault:compress to save a session log
- When a design decision is made, run /vault:spec to document it
- When a product requirement is defined, run /vault:prd to document it
- Documents written for this project (meeting prep, question lists, research, proposals, working notes) go in the vault under docs/ in the matching type folder (docs/prep/, docs/research/, docs/notes/), never at the vault project root
```

The sections this template owns are `## Context`, `## About this project`, `## Vault`, `## Code folder`, and `## Rules`. When a `CLAUDE.md` already exists, a skill replaces only these and keeps every other section intact, in place: other tools write their own sections into the same file.
