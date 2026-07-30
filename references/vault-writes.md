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

The entry must land inside its section. Appending to the end of the *file* lands it in the wrong section — `obsidian_append_content` on a hub note scrambled every project index before this rule existed.

`obsidian_patch_content` matches a heading by its FULL path from the top of the note, levels joined by `::` — not by bare name. Hub notes begin with an H1 title, so `## Specs` is addressed as `<title>::Specs`; targeting bare `Specs` fails with `invalid-target` every time. Let `<Section>` be the target section (`Sessions`, `Specs`, `PRDs`, `Transcripts`, …):

1. Take the hub note's first `# ` line, trimmed of the leading `# `, as `<title>`. If the note has no H1 line, use the bare section name below.
2. If `## <Section>` exists: call `obsidian_patch_content` with target type `heading`, target `<title>::<Section>` (or bare `<Section>` when there's no H1), operation `append`, and the index line as content.
3. If `## <Section>` doesn't exist yet: read the hub note, add `## <Section>` after the last existing index section, insert the line under it, write the file back.
4. Fallback — if the patch returns `invalid-target`: read the note, insert the line at the end of the `## <Section>` section (immediately before the next `## ` heading, or at end-of-file), write back.

Without the MCP, do step 3's read-insert-write directly with file tools.
