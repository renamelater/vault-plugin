# Claude Navigation Instructions

You are my AI second brain assistant. Always read this file first before doing anything else.

## Who I am

<!-- One line about you: role, what you work on, how you like to communicate. -->

## Vault Structure

```
vault/
├── CLAUDE.md                        ← you are here
├── context/                         ← always read first, applies to everything
│   ├── about-me.md                  ← who I am, values, background
│   └── preferences.md               ← how I like to work and communicate
├── work/
│   ├── projects/                    ← active work projects
│   │   └── [project]/
│   │       ├── [project].md         ← hub note (named after the project)
│   │       ├── sessions/            ← session logs for this project
│   │       └── docs/
│   │           ├── specs/           ← design decisions and specs
│   │           ├── prds/            ← product requirements documents
│   │           └── [type]/          ← ingested content (see /vault:ingest types)
│   ├── general/                     ← work content with no project (rare)
│   └── tasks/
│       ├── current.md               ← active work to-dos
│       └── archive.md               ← completed to-dos (moved here, never deleted)
├── personal/
│   ├── projects/                    ← active personal projects (same shape as work)
│   ├── interests/                   ← ongoing topic notes, no sessions/docs
│   ├── general/                     ← personal content with no project (rare)
│   └── tasks/
│       ├── current.md               ← active personal to-dos
│       └── archive.md               ← completed personal to-dos
└── resources/                       ← reusable prompts and references
```

## Hub Notes

Every project folder has a **hub note named after the project** (e.g. `my-project.md`), NOT `README.md` — this makes `[[project-name]]` links resolve in Obsidian. The hub note holds: what the project is, Status, Started date, Code folder path, notes, and the index sections (`## Sessions`, `## Specs`, `## PRDs`, plus type sections like `## Transcripts`).

- **Status values:** `Active`, `Paused`, or `Done`. "What should I focus on" only scans Active projects.

## Navigation Rules

1. Read `context/` files before any task — they apply universally
2. For work tasks → navigate into `work/`; for personal tasks → `personal/`; when unsure, check both
3. **When starting work in a project:** read the relevant `tasks/current.md`, and if any open items relate to this project, surface the 2–3 most relevant ones up front. Don't wait to be asked.

### Where to save things

| Type | Location |
|------|----------|
| Session logs | `[bucket]/projects/[project]/sessions/YYYYMMDD-HHMM-title.md` |
| Specs (design decisions) | `[bucket]/projects/[project]/docs/specs/YYYYMMDD-HHMM-title.md` |
| PRDs | `[bucket]/projects/[project]/docs/prds/YYYYMMDD-HHMM-title.md` |
| Ingested content | `[bucket]/projects/[project]/docs/[type-folder]/YYYYMMDD-HHMM-title.md` — type-folder: `transcripts/`, `research/`, `articles/`, `interviews/`, `brainstorms/`, `feedback/`, `prep/`, `notes/` |
| Self-authored working docs (meeting prep, question lists, research, proposals) | `[bucket]/projects/[project]/docs/[type-folder]/` — same taxonomy as ingested content; living documents that update across sessions keep a plain name with no timestamp prefix. NEVER at the project root |
| Non-project content | `[bucket]/general/[type-folder]/` |
| Active tasks | `[bucket]/tasks/current.md` |
| Completed tasks | `[bucket]/tasks/archive.md` (move, don't delete) |
| Reusable templates | `resources/` |

`[bucket]` = `work` or `personal`. One term for decisions: **specs** — a design or product decision is a spec.

### When I say "remember this"

Identify the category from the table above, save it to the correct file immediately, confirm with the file path.

### When I say "what should I focus on"

Read both `tasks/current.md` files, check the most recent session file across Active projects, synthesize a prioritized response.

## Skills

| Command | When to use |
|---------|-------------|
| `/vault:setup` | First-time vault setup (you've likely already run this) |
| `/vault:new-project` | Setting up a new work or personal project |
| `/vault:relink` | Point an existing code folder at an existing vault project |
| `/vault:compress` | End of every session — saves a session log, reconciles action items |
| `/vault:spec` | A design decision was made — document it |
| `/vault:prd` | A product requirement is defined — document it |
| `/vault:search-sessions` | Looking up what was done in a past session |
| `/vault:ingest` | Save external content (transcripts, research, articles…) into the project |

**Note on `/vault:new-project` and `/vault:relink`:** these two are user-only (`disable-model-invocation`) — Claude cannot see or invoke them, and they won't appear in Claude's skill list. They only run when the user sends the command as its own message. If the user mentions one mid-sentence, it did not fire and Claude cannot fire it — don't claim the command doesn't exist; instead say it's user-only and ask them to type the bare `/vault:...` command.

## Linking Rules

Notes are connected with Obsidian's `[[filename]]` link syntax (filename without `.md`). The skills create links automatically — follow the same rules when saving manually:

- **Sessions** link to any specs or PRDs saved during that session, and to the previous session via the `previous:` frontmatter field
- **PRDs** link to any related specs
- **Ingested content** links to related sessions, specs, or PRDs mentioned in the content
- **Projects** are linked by hub note name: `[[my-project]]`
- **Hub note index sections** — entries are ONE line: `- [[filename]] — descriptor (8 words max)`. Insert under the correct heading, never append to the end of the file.
