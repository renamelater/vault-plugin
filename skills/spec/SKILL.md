---
name: spec
description: Save a design decision into the Obsidian vault as a spec. Use when the user says "spec", or a design decision has just been settled in conversation.
---

# Save Spec / Design Decision

When invoked, do the following:

## Step 1: Determine the Vault Project (in order)

Follow `../../references/resolve-project.md` (relative to this skill's folder) to resolve the **bucket** and **project slug** — read that file now, do not guess the project. If it resolves, continue to Step 2. If no match:

### No match found
Stop and tell the user:
- "I don't see a project for this spec. Two options:
   1. Run `/vault:new-project` first to set one up, then re-run `/vault:spec`
   2. Use an existing project I missed — tell me which one"
- **Never invoke `/vault:new-project` from this skill** — that's the user's call to make explicitly
- If they pick option 2, use the project they named and continue

## Step 2: Find the Hub Note
Locate the project's **hub note** (defined in the shared reference from Step 1).

## Step 3: Ask What to Document
If the user hasn't already provided the content, ask:
- "What's the decision or spec to document?"

Keep it to one question. If context is already clear from the conversation, skip asking and use that context directly.

## Step 4: Create the Spec Entry
Write a markdown file with this structure:

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

Note: don't add a link to today's session log — at spec time it usually doesn't exist yet. `/vault:compress` links session → spec at the end of the session; that single direction is the source of truth.

## Step 5: Save to Vault

Before constructing the filename or the `date:` field, read `../../references/vault-writes.md` and follow its Timestamps and Frontmatter rules — it also owns the hub-note insertion procedure used in Step 6.

- Use `obsidian_append_content` to save the file
- Target path: `[bucket]/projects/[project-slug]/docs/specs/YYYYMMDD-HHMM-short-title.md`
  - Example: `work/projects/selective-notification/docs/specs/20260413-1415-notification-states.md`

## Step 6: Update the Hub Note

- Add an **index line** for the new spec under `## Specs`, following the insertion procedure in `vault-writes.md` exactly.
- If the hub note has a `## Decisions` section instead of `## Specs` (older projects), use it and rename the heading to `## Specs` while you're in the file.

## Step 7: Confirm
- Tell the user the spec was saved
- Show the exact file path

## Rules
- Be concise — capture enough to understand the decision later without over-documenting
- If the user provides the content, don't ask questions — just format and save it
- Omit any sections that aren't relevant to the specific decision
