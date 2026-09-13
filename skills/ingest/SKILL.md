---
name: ingest
description: Ingest external content into the Obsidian vault. Use when the user says "ingest" or "save this to the vault", pastes external content (transcript, research, article, interview) without other instructions, or shares a Notion URL.
---

# Ingest External Content into the Vault

## Step 1: Read the Content

- The user will paste content directly into the conversation, or attach/reference a file (read attached files from wherever the current environment surfaces them)
- **Notion URL?** When the content is a Notion URL (`notion.so`, `*.notion.site`, `app.notion.com`), read `${CLAUDE_PLUGIN_ROOT}/skills/ingest/notion.md` and `${CLAUDE_PLUGIN_ROOT}/references/notion.md` together in one turn, then follow them — the first owns fetching and the shape gate and says where to rejoin these steps, the second owns Notion tool access and the no-tools branch
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

The project `CLAUDE.md` in cwd is already in context. Its `## Vault` section's `Overview:` line is the whole resolution:

`Overview: [vault]/personal/projects/portfolio-site-mw/portfolio-site-mw.md`

- **hub note** — that path, verbatim
- **vault root** — everything before the `/work/` or `/personal/` segment
- **bucket** — that segment. **slug** — the segment after `projects/`

**Sanity-check the match.** When the content's subject clearly belongs to a different vault project than the resolved one (team content ingested from an unrelated repo, a Notion URL pasted mid-task), name both projects and confirm the target before filing.

No `Overview:` line, or no project `CLAUDE.md` in cwd: read `${CLAUDE_PLUGIN_ROOT}/references/resolve-project.md` — it owns the resolution ladder, the tool map for setups without Route A, and no-match handling. Its no-match list takes two more options here: route to `work/general/[type-folder]/` or `personal/general/[type-folder]/` (content with no project). On either, the target folder is that path directly, with no `docs/` nesting.

**Target folder** otherwise: `[bucket]/projects/[slug]/docs/[folder-name]/`, the canonical folder name from Step 2, nested under `docs/` alongside `specs/` and `prds/`. The canonical name is the folder whether or not it exists yet: one content type, one folder. Saving the file creates the path on write; there is no separate create step.

## Step 4: Gather — One Turn

Everything below is independent of everything else. Issue it all in a single turn:

1. `date '+%Y%m%d-%H%M %Y-%m-%d %H:%M'` — the filename stamp and the frontmatter stamp. Never estimate either from conversation context; the model's internal clock drifts by hours.
2. `Read ${CLAUDE_PLUGIN_ROOT}/skills/ingest/types.md` — what to extract for the type from Step 2.
3. `Read ${CLAUDE_PLUGIN_ROOT}/references/vault-writes.md` — the insertion procedure Step 6 follows.
4. `vault_get_document_map` on the hub note — the `version` token for the Step 6 patch, and whether the type's section already exists.
5. `vault_list` the target folder — a "not found" here is the answer, not a failure: it means Step 7 reports the folder as newly created.
6. `vault_read` `[bucket]/tasks/current.md` — only if the content carries action items assigned to the user.
7. **The link searches**, below.

### The link searches

Pull **identifiers** out of the content: ticket IDs (`LHA-3917`), proper nouns, product and feature names, document titles, people. **Five at most, ranked by how specific they are** — an identifier that would match one note is worth searching; a common word is not. Skip anything that reads like ordinary prose.

Scope every search to the project, so a common token can't return the whole vault:

```json
{"and": [
  {"regexp": ["^work/projects/selective-notification/", {"var": "path"}]},
  {"regexp": ["LHA-3917", {"var": "content"}]}
]}
```

`search_query` with that shape, one call per identifier, all in this turn. It answers in paths, not excerpts — which is the point, since a common token through `search_simple` can return tens of kilobytes of context and sit in the session for the rest of it. Vault filenames are descriptive enough to judge relevance; `vault_read` the one or two that stay ambiguous. On Route B or no MCP, `obsidian_complex_search` / Grep restricted to the same folder. Widen beyond the project only when the user asks for it.

This step is done when **every identifier on the ranked list has been searched** — not when the first few links are found.

## Step 5: Compose the Ingest File

**The snippet rule:** preserve verbatim only what's worth referencing later — direct quotes capturing decisions, opinions, or precise phrasing; specific numbers, dates, names, or commitments; nuances a summary would flatten. Skip filler, restatements, and noise. Never save the entire raw content — this is a curated ingest, not an archive.

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
<Links from the Step 4 searches — verify each hit actually relates before linking>
- [[filename-without-extension]]

## Open Questions
<Anything unresolved or worth following up on>
```

Any frontmatter value containing a colon, quote, bracket, or `#` goes in single quotes (double any single quotes inside) — `source:` is the usual offender, since document titles carry colons.

Under `## Related`: link the project hub note as `[[slug]]` (hub notes are named after their project, so this resolves), plus the specs, PRDs, and sessions the searches surfaced. Search hits aren't automatic links — verify each one actually relates.

## Step 6: Save and Index — One Turn

Different files, so every write goes in the same turn:

1. `vault_append` the note to `[target-folder]/YYYYMMDD-HHMM-short-title.md`
   - Example: `work/projects/selective-notification/docs/transcripts/20260421-1430-q2-design-review.md`
2. `vault_patch` the index line under the hub-note section matching the content type (`## Transcripts`, `## Research`, `## Prep`, …), per the procedure in `vault-writes.md` — including its missing-section case. On Route A the `ifMatch` and `rejectIfContentPreexists` guards make the call its own confirmation — a wrong or duplicated write fails loudly rather than landing silently, so don't re-read the hub to check.
3. `vault_patch` the user's own action items into `[bucket]/tasks/current.md`, as `- [ ] <action> (from [[YYYYMMDD-HHMM-short-title]]) — due <date if specified>`. Only the user's items; never items assigned to others. No actionable items: skip.

## Step 7: Confirm

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
