---
name: compress
description: Compress the session into the Obsidian vault. Use when the user says "compress" or "save session", or the session is wrapping up.
---

# Compress & Save Session to Obsidian

## Step 1: Resolve the Vault Project

The project `CLAUDE.md` in cwd is already in context. Its `## Vault` section's `Overview:` line is the whole resolution:

`Overview: [vault]/personal/projects/portfolio-site-mw/portfolio-site-mw.md`

- **hub note** — that path, verbatim
- **vault root** — everything before the `/work/` or `/personal/` segment
- **bucket** — that segment. **slug** — the segment after `projects/`

No `Overview:` line, or no project `CLAUDE.md` in cwd: read `${CLAUDE_PLUGIN_ROOT}/references/resolve-project.md` — it owns the resolution ladder, the tool map for setups without Route A, and no-match handling. Its no-match list takes a third option here: `/vault:relink` to point this folder at an existing vault project.

Tell the user where this is going in one line ("Saving to `personal/projects/portfolio-site-mw/sessions/`") so a wrong match gets caught before anything is written.

## Step 2: Gather — One Turn, Five Calls

Everything the note needs is independent. Issue all of it in a single turn:

1. `date '+%Y%m%d-%H%M %Y-%m-%d %H:%M'` — the filename stamp and the frontmatter stamp. Never estimate either from conversation context; the model's internal clock drifts by hours.
2. `vault_read` the hub note, untargeted — the **newest entry under `## Sessions` names the `previous:` link**, and the file carries its own H1 and `## Status`. Go by the `YYYYMMDD-HHMM` stamp that leads each filename, since an older hub can list entries out of order. The index already names the file, so the `sessions/` folder stays unlisted.
3. `vault_get_document_map` on the hub note — the `version` token for the Step 4 patch.
4. `vault_read` `[bucket]/tasks/current.md` — open items to reconcile in Step 4.
5. `Read ${CLAUDE_PLUGIN_ROOT}/references/vault-writes.md` — the insertion procedure Step 4 follows.

Also collect, from conversation history alone: any `/vault:spec` or `/vault:prd` filenames saved this session, for the `## Specs & PRDs` section.

## Step 3: Compose the Session Note

```
---
type: claude-code-session
project: <project-slug>
date: <YYYY-MM-DD HH:mm>
previous: "[[<filename from the newest ## Sessions entry>]]"
tags:
  - claude-code
  - <project-slug>
---

# <Brief title describing what was accomplished>

## Summary
<2-4 sentence summary of what was discussed and accomplished>

## Key Decisions
- <List important decisions made>

## Changes Made
- <List files created, modified, or deleted>

## Problems Solved
- <List any bugs fixed or issues resolved>

## Open Items
- <List anything left unfinished or to follow up on>

## Specs & PRDs
<Only include this section if specs or PRDs were saved this session>
- [[filename-of-spec-or-prd]]

## Session Log
<Condensed version of key conversation exchanges — keep only the important parts, skip tool outputs and verbose details>
```

Omit `previous:` if this is the project's first session. Any frontmatter value containing a colon, quote, bracket, or `#` goes in single quotes — titles are the usual offender.

## Step 4: Save, Index, Reconcile — One Turn

Every write below targets a different file, so they all go in the same turn:

1. `vault_append` the note to `[bucket]/projects/[slug]/sessions/YYYYMMDD-HHMM-short-title.md`. A session log records the *activity*, so it lands in `sessions/` whatever the session was about — including one spent entirely on spec or PRD work.
2. `vault_patch` the index line under the hub note's `## Sessions`, per the procedure in `vault-writes.md`. On Route A the `ifMatch` and `rejectIfContentPreexists` guards make the call its own confirmation — a wrong or duplicated write fails loudly rather than landing silently, so go straight to Step 5 rather than re-reading the hub.
3. **Task reconciliation**, for the items in `tasks/current.md` this session **clearly completed**: `vault_patch` them into `[bucket]/tasks/archive.md` under a `## YYYY-MM-DD` heading, each as `- [x] <item> (completed YYYY-MM-DD, see [[session-file]])`, then drop them from `current.md` — one `vault_patch` with `operation: replace`, `within` picking the list block, carrying that list rewritten without them. (`delete` removes a whole block, so it is the wrong tool for a single line.) Clear-cut items only; anything **ambiguous** stays put and goes to Step 5. Nothing in the list matches this project: skip silently.

## Step 5: Confirm

Everything is saved by now, so any question here holds up nothing:

- The exact file path, and that the hub note was indexed
- Open items worth following up on, and anything archived in Step 4
- **Ambiguous task items** — ask in ONE line listing the candidates: "These look done — archive? 1) … 2) …"
- If the hub note's `## Status` is not `Active`, ask in the same breath whether to flip it back

## Rules
- Strip verbose tool outputs, stack traces, and repetitive back-and-forth
- Focus on WHAT was done, WHY, and any key code or commands worth remembering
