---
name: new-project
description: Set up a new work or personal project in the vault.
disable-model-invocation: true
---

# New Project Setup

When invoked, do the following:

Vault root, `[vault]` substitution, and file-access rules: `../../references/resolve-project.md` — read it before touching the vault.

## Step 1: Ask Four Questions
Ask all four at once:

1. "Work or personal?"
2. "What's the project name?" (use lowercase-with-hyphens format — this names the **vault** folder, not the code folder)
3. "One line — what is this project?"
4. "Code project or knowledge-only?" (code = there's a code folder Claude works in, e.g. a Cursor project; knowledge-only = research/notes live entirely in the vault, e.g. home-theater)

## Step 2: Get the Real Current Time

**DO NOT guess the date.** Run this exact bash command and use its output verbatim for the `Started` field:

```bash
date '+%Y-%m-%d'
```

## Step 3: Create Vault Structure
Based on the answers, use `obsidian_append_content` to create the following files:

- `[work or personal]/projects/[project-name]/[project-name].md`  ← the **hub note**, named after the project so `[[project-name]]` links resolve. NEVER name it `README.md`.
- `[work or personal]/projects/[project-name]/sessions/.keep.md`
- `[work or personal]/projects/[project-name]/docs/specs/.keep.md`
- `[work or personal]/projects/[project-name]/docs/prds/.keep.md`

### Hub note content:
```
# [Project Name]

## What is this
[one line description from Step 1]

## Status
Active

## Started
[YYYY-MM-DD]

## Code
[absolute path of cwd — code projects only; omit this section for knowledge-only projects]

## Notes

---

## Sessions
<!-- /vault:compress inserts here -->

## Specs
<!-- /vault:spec inserts here -->

## PRDs
<!-- /vault:prd inserts here -->
```

## Step 4: Create CLAUDE.md in the Current Working Directory (code projects only)

**Knowledge-only projects: skip this step entirely** — there's no code folder, and the vault folder is the whole project. Confirm and finish.

For code projects, create a `CLAUDE.md` at `./CLAUDE.md` (the current working directory) using direct file creation (not Obsidian MCP — this file lives in the code project, not the vault).

This skill assumes you're running it from inside Cursor (or another editor) with the code project folder already open. The CLAUDE.md should land in the folder you're currently in — DO NOT construct a path from the project name slug, and DO NOT create a new folder.

**Guardrail — existing CLAUDE.md:** if `./CLAUDE.md` already exists, do NOT overwrite it. Tell the user what it currently points at and ask whether to (a) add/replace only the vault sections (`## Context`, `## About this project`, `## Vault`, `## Code folder`, `## Rules`) while preserving everything else in the file — other tools like Impeccable write their own sections into CLAUDE.md — or (b) leave it alone. If the folder is already linked to another vault project, suggest `/vault:relink` instead.

**Guardrails — before writing, check the current working directory:**
- If cwd is `[vault]` or anywhere inside it → STOP. Tell the user: "It looks like you're running this from the Obsidian Vault, not your code project. Open your code folder in Cursor and run /vault:new-project from there." Skip this step entirely.
- If cwd is the home directory → STOP with the same warning.
- Otherwise → write `CLAUDE.md` to the current working directory.

### CLAUDE.md content:
```
# [Project Name]

## Context
Full project context lives in the Obsidian vault:
`[vault]/[work or personal]/projects/[project-name]/`

Always read the vault hub note before starting work.
Always read the most recent file in the vault sessions/ folder to see where we left off.

## About this project
[one line description]

## Vault
- Overview: [vault]/[work or personal]/projects/[project-name]/[project-name].md
- Sessions: [vault]/[work or personal]/projects/[project-name]/sessions/
- Specs: [vault]/[work or personal]/projects/[project-name]/docs/specs/
- PRDs: [vault]/[work or personal]/projects/[project-name]/docs/prds/

## Code folder
[absolute path of cwd]

## Rules
- At the end of every session, run /vault:compress to save a session log
- When a design decision is made, run /vault:spec to document it
- When a product requirement is defined, run /vault:prd to document it
- Documents written for this project (meeting prep, question lists, research, proposals, working notes) go in the vault under docs/ in the matching type folder (docs/prep/, docs/research/, docs/notes/), never at the vault project root
```

## Step 5: Confirm
Tell the user:
- Vault folder created at: `[path]`, hub note `[project-name].md`
- CLAUDE.md created at: `[cwd]/CLAUDE.md` (code projects)
- Remind them to run `/vault:compress` at the end of each session

## Rules
- The hub note is ALWAYS `[project-name].md`, never `README.md` — Obsidian wikilinks resolve by filename, and `[[project-name]]` must point at the hub
- Both the vault scaffold and (for code projects) the CLAUDE.md must succeed; if one fails, tell the user which one
