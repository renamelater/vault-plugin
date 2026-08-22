---
name: ingest
description: Ingest external content into the Obsidian vault. Use when the user says "ingest" or "save this to the vault", pastes external content (transcript, research, article, interview) without other instructions, or shares a Notion URL.
---

# Ingest External Content into the Vault

When invoked, do the following:

## Step 1: Read the Content

- The user will paste content directly into the conversation, or attach/reference a file (read attached files from wherever the current environment surfaces them)
- **Notion URL?** When the content is a Notion URL (`notion.so`, `*.notion.site`, `app.notion.com`), read `notion.md` in this skill's folder and follow it — it owns fetching and the shape gate, and says where to rejoin these steps
- If the content isn't obvious in the current message, ask: "What would you like to ingest?"
- Read the full content before classifying

## Step 2: Classify the Content Type

Identify what kind of content this is. Use these **canonical type names** — never invent alternates like `transcript` vs `transcripts` or `meeting` vs `meetings`:

| Signals | Canonical type | Folder name |
|---------|----------------|-------------|
| Speaker labels, timestamps, dialogue, meeting-style back-and-forth | transcript | `transcripts/` |
| Cited sources, structured findings, literature review | research | `research/` |
| Single-author prose, headline, published format | article | `articles/` |
| Q&A format, one interviewer + interviewee | interview | `interviews/` |
| Exploratory thoughts, mind-map style, ideation | brainstorm | `brainstorms/` |
| Product/feature feedback, user reports | feedback | `feedback/` |
| Agenda, discussion points, questions to bring, pre-meeting or pre-call briefing | prep | `prep/` |
| Anything else | note | `notes/` |

**Classification approach:** make your best guess and tell the user what you picked in the confirmation step. If the user disagrees, they'll correct and you'll move the file.

## Step 3: Resolve the Vault Project

Read `../../references/resolve-project.md` (relative to this skill's folder) before touching the vault: it owns the vault root and `[vault]` substitution, project resolution (**bucket** and **slug**), file placement, tool access, and no-match handling. Resolve the bucket and slug with it now. If they resolve, sanity-check the match before continuing: when the content's subject clearly belongs to a different vault project than the resolved one (team content ingested from an unrelated repo, a Notion URL pasted mid-task), name both projects and confirm the target before filing. Then continue to Step 4.

No match (or the cwd isn't a project directory): follow its no-match handling, adding two options, route to `work/general/[type-folder]/` or to `personal/general/[type-folder]/` (content with no project). On either, Step 4 targets that folder directly with no `docs/` nesting.

## Step 4: Determine or Create the Target Folder

- Target folder: `[project-path]/docs/[folder-name]/`, the canonical folder name from Step 2, nested under `docs/` alongside `specs/` and `prds/`. The canonical name is the folder whether or not it exists yet: one content type, one folder.
- Use `obsidian_list_files_in_vault` to note whether the folder already exists (Step 12 reports new folders). Saving the file creates the path on write; there is no separate create step.

## Step 5: Read the Write Rules

Read `../../references/vault-writes.md` now and follow it for the rest of this run — it owns timestamps (never guess the time), frontmatter YAML quoting, and the hub-note insertion procedure used in Step 11.

## Step 6: Extract Structure and Snippets

Read `types.md` in this skill's folder and extract what it lists for the type from Step 2.

**The snippet rule:** Preserve verbatim only what's worth referencing later — direct quotes capturing decisions, opinions, or precise phrasing; specific numbers, dates, names, or commitments; nuances a summary would flatten. Skip filler, restatements, and noise. Never save the entire raw content — this is a curated ingest, not an archive.

## Step 7: Write the Ingest File

Use this structure (omit sections that don't apply):

```
---
type: <canonical-type>
project: <project-name>
date: <YYYY-MM-DD HH:mm>
source: <optional — URL, meeting name, author, etc.>
attendees: [<only for transcripts/interviews>]
tags:
  - <canonical-type>
  - <project-name>
---

# <Brief title describing the content>

## Summary
<2–4 sentence summary of the content>

## Key Points / Decisions
- <Bulleted list of the most important takeaways>

## Action Items
<Only if the content contains actionable items>
- [ ] <Owner> — <action> by <date>

## Key Quotes
<Direct quotes worth preserving verbatim>
> "<exact quote>"
> — <speaker or source>, <context>

## Notable Snippets
<Longer passages worth preserving, with context>
On <topic>:
> <verbatim passage>

## Related
<Links to sessions, specs, PRDs, or other notes — see Step 9>
- [[filename-without-extension]]

## Open Questions
<Anything unresolved or worth following up on>
```

## Step 8: Save to the Vault

- Use `obsidian_append_content` to save the file
- Target path: `[project-path]/docs/[folder-name]/YYYYMMDD-HHMM-short-title.md`
  - Example: `work/projects/selective-notification/docs/transcripts/20260421-1430-q2-design-review.md`
- Verify the path uses the canonical folder name from Step 2 and nests under `docs/`

## Step 9: Auto-Link Related Notes

**Search the vault — don't rely on memory.** Pull key terms from the content (ticket IDs like LHA-3917, feature names, people, project names) and run `obsidian_simple_search` on each to find candidate related notes. Then:

- Project mentions → link the project hub note using `[[project-slug]]` (hub notes are named after their project, so this resolves)
- Specs or PRDs surfaced by search → link using `[[filename-without-extension]]`
- Sessions surfaced by search → link if clearly related
- Verify each candidate actually relates before linking — search hits aren't automatic links

Add these under the `## Related` section using Obsidian `[[wikilink]]` syntax. This step is done when **every extracted key term has been searched** — not when the first few links are found.

## Step 10: Extract Action Items to Tasks

- If the content contains action items assigned to the user, append them to `/work/tasks/current.md` (or `/personal/tasks/current.md` for personal content)
- Format: `- [ ] <action> (from [[YYYYMMDD-HHMM-short-title]]) — due <date if specified>`
- Only the user's action items go into their task list — don't add items assigned to others
- Skip this step if there are no actionable items

## Step 11: Update the Hub Note

Add an **index line** for the ingested file under the hub-note section matching the content type (`## Transcripts`, `## Research`, `## Prep`, …), following the insertion procedure in `vault-writes.md` exactly — including its missing-section case.

## Step 12: Confirm

Tell the user:
- The detected content type (e.g., "Classified as a transcript")
- The exact file path where it was saved
- Any new folders created
- Any action items added to their tasks
- Any related notes auto-linked
- Offer: "If the classification is wrong, tell me the correct type and I'll move it."

## Rules

- **One file per ingest** — don't save a separate raw file. Important verbatim passages live inline in the `## Key Quotes` and `## Notable Snippets` sections.
- **Curate, don't archive** — preserve signal, drop noise. A year of ingested transcripts shouldn't bloat the vault.
