---
name: compress
description: Compress the session into the Obsidian vault. Use when the user says "compress" or "save session", or the session is wrapping up.
---

# Compress & Save Session to Obsidian

When invoked, do the following:

## Step 1: Determine the Vault Project (in order)

Follow `../../references/resolve-project.md` (relative to this skill's folder) to resolve the **bucket** and **project slug** — read that file now, do not guess the project. If it resolves, continue to Step 2. If no match:

### No match found
Stop and tell the user:
- "I can't find a vault project for this folder. Three options:
  1. Run `/vault:new-project` to set one up, then re-run `/vault:compress`
  2. Run `/vault:relink` to point this folder at an existing vault project
  3. Tell me which existing vault project to save under"
- **Never invoke `/vault:new-project` or `/vault:relink` from this skill** — those are the user's calls to make explicitly
- If they pick option 3, use the project they name and continue

## Step 2: Confirm the Target
Briefly tell the user which vault project this session will be saved under (e.g. "Saving to `personal/projects/portfolio-site-mw/sessions/`") so they can catch a wrong match before anything is written.

## Step 3: Find the Hub Note
Locate the project's **hub note** (defined in the shared reference from Step 1). Every "hub note" mention below means this file.

## Step 4: Collect Specs and PRDs Saved This Session
- Look at the conversation history for any `/vault:spec` or `/vault:prd` commands that were run
- Note the filenames of any specs or PRDs saved (without `.md` extension)
- These will be linked in the session log

## Step 5: Reconcile Action Items
- Read `[bucket]/tasks/current.md`
- Find open items related to this project
- For each item this session **clearly completed**: check it off and move it to `[bucket]/tasks/archive.md` under a `## YYYY-MM-DD` heading, as `- [x] <item> (completed YYYY-MM-DD, see [[session-file]])`
- If it's **ambiguous** whether an item was completed, ask the user in ONE line listing the candidates: "These look done — archive? 1) … 2) …"
- If nothing matches this project, skip silently — don't interrogate

## Step 6: Create Session Summary

First, list the project's `sessions/` folder and note the most recent existing session file — it becomes the `previous:` link (omit the field if this is the first session).

Write a concise markdown note with this structure:

```
---
type: claude-code-session
project: <project-slug>
date: <YYYY-MM-DD HH:mm>
previous: "[[<filename of most recent prior session>]]"
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

## Step 7: Save to Sessions Folder

**Path guardrail — READ BEFORE WRITING:** This skill ALWAYS writes to the project's `sessions/` subfolder. NEVER write to `docs/prds/` or `docs/specs/`, even if the session was primarily about PRD or spec work. Session logs document the *activity*; specs and PRDs document the *artifacts*. If the target path you are about to use contains `/docs/prds/` or `/docs/specs/`, stop and switch it to `/sessions/`.

Before constructing the filename or the `date:` field, read `../../references/vault-writes.md` and follow its Timestamps rule — it also owns the hub-note insertion procedure used in Step 8.

- Use `obsidian_append_content` to save the note
- Target path: `[bucket]/projects/[project-slug]/sessions/YYYYMMDD-HHMM-short-title.md`
  - Example: `work/projects/selective-notification/sessions/20260413-1045-notification-detail-layout.md`
- Before writing, verify the path contains `/sessions/` and NOT `/docs/`

## Step 8: Update the Hub Note

- Add an **index line** for the new session under `## Sessions`, following the insertion procedure in `vault-writes.md` exactly.
- While the hub note is open: if `## Status` is not `Active`, ask the user in one line whether to flip it back to Active.

## Step 9: Confirm
- Tell the user the session was saved
- Show the exact file path
- Call out any open items to follow up on, and any action items archived in Step 5

## Rules
- Keep summaries concise but useful for future reference
- Include enough detail that someone reading it later understands what happened
- Strip verbose tool outputs, stack traces, and repetitive back-and-forth
- Focus on WHAT was done, WHY, and any key code or commands worth remembering
