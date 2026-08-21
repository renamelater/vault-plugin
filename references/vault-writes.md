# Vault Writes: Timestamps, Frontmatter & the Hub-Note Index

Rules for every skill that writes files into the vault. Read this before saving anything.

## Timestamps

Never estimate the date or time from conversation context — the model's internal clock drifts by hours. Run exactly:

```bash
date '+%Y%m%d-%H%M %Y-%m-%d %H:%M'
```

Two values on one line: the `YYYYMMDD-HHMM` stamp for the filename, and the `YYYY-MM-DD HH:MM` stamp for the frontmatter `date:` field. Use both verbatim (the command uses the system timezone).

## Frontmatter YAML

Any frontmatter value containing a colon, quotes, brackets, or `#` MUST be wrapped in single quotes (double any single quotes inside it). An unquoted colon breaks YAML parsing and Obsidian renders the whole block as raw red text instead of Properties. The `source:` field is the usual offender — document titles contain colons.

## Index lines

An **index line** is the one-line entry a skill adds to a hub-note section:

`- [[YYYYMMDD-HHMM-short-title]] — <descriptor, 8 words max>`

The descriptor is a glanceable label, not a summary — full detail lives in the linked file.

## Inserting an index line into the hub note

The entry must land **inside its section**. Appending to the end of the *file* lands it in the wrong section: a plain whole-file append on a hub note scrambled every project index before this rule existed. So never use `obsidian_append_content` / `vault_append` / a shell `>>` on a hub note.

Every route below does the same thing: address the `## <Section>` heading, append the index line to that heading's body. Pick the route matching your setup (`resolve-project.md` has the tool map). Let `<Section>` be `Sessions`, `Specs`, `PRDs`, `Transcripts`, `Prep`, `Research`, `Notes`, and so on.

### Route A: REST API built-in server (`vault_patch`)

The target is an **array of heading texts**, not a `::`-joined string. A bare string is rejected. The array is the full path from the top of the document down, and hub notes open with an H1 title, so the H1 is the first element:

1. Call `vault_get_document_map` on the hub note. It returns the heading tree and a `version` token. The tree nests sections under the H1, e.g. `{"Vault Plugin": {"Sessions": {}, "Specs": {}}}`, which makes the path for `## Sessions` exactly `["Vault Plugin", "Sessions"]`.
2. Call `vault_patch`:

```json
{
  "path": "personal/projects/vault-plugin/vault-plugin.md",
  "targetType": "heading",
  "target": ["Vault Plugin", "Sessions"],
  "operation": "append",
  "within": -1,
  "content": "\n- [[20260821-1631-short-title]] index descriptor here",
  "ifMatch": "<version from the document map>",
  "rejectIfContentPreexists": true
}
```

Notes that matter (each verified with a live write, 2026-08-21):
- `within: -1` plus the leading `\n` in `content` is what keeps the index a **contiguous list**. It splices onto the section's last block, so the new line lands directly under the previous entry. Without `within`, the engine inserts the line as a new block with a blank line above it, and the index double-spaces by one entry per write.
- `scope` defaults to `content`, the heading's body, which is what you want. Do not pass `markerAndContent`: that appends a sibling *section*, not a line.
- `ifMatch` makes the write fail with `ifMatch precondition failed` if the note changed since step 1, and nothing is written. Always pass it; it costs one extra call.
- `rejectIfContentPreexists` makes a retry idempotent: if the line is already in the section, the call fails with `the target already contains the content to append` and nothing is duplicated. It combines with `within`.
- Copy heading keys **verbatim** from the document map. A repeated heading name carries a non-printable marker suffix; retyping it silently targets the wrong one.

### Route B: mcp-obsidian (`obsidian_patch_content`)

Here the heading path is a **string joined by `::`**, not an array. Hub notes begin with an H1 title, so `## Specs` is addressed as `<title>::Specs`; targeting bare `Specs` fails with `invalid-target` every time.

1. Take the hub note's first `# ` line, trimmed of the leading `# `, as `<title>`.
2. Call `obsidian_patch_content` with target type `heading`, target `<title>::<Section>` (or bare `<Section>` when the note has no H1), operation `append`, and the index line as content.

### Route C: no MCP

Read the note, insert the line at the end of the `## <Section>` section (immediately before the next `## ` heading, or at end-of-file), write the file back. Never a bare append.

### If the section does not exist yet

Route A can pass `createTargetIfMissing: true`, but that places the heading without control over where. Prefer the explicit path on every route: read the hub note, add `## <Section>` after the last existing index section, insert the line under it, write back.

### Verify

After any route, re-read the section and confirm the line is inside it and the neighbouring sections are untouched. This is a two-second check that catches the failure this whole rule exists to prevent.
