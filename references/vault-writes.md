# The Hub-Note Index Line

How a skill adds its entry to a hub note. Read from the gathering step of whichever skill is running, alongside its other opening calls.

Timestamps and frontmatter quoting are stated inline in each skill, so they are not repeated here.

## Index lines

An **index line** is the one-line entry a skill adds to a hub-note section:

`- [[YYYYMMDD-HHMM-short-title]] — <descriptor, 8 words max>`

The descriptor is a glanceable label, not a summary — full detail lives in the linked file.

## Inserting it

The entry must land **inside its section**. Appending to the end of the *file* lands it in the wrong section: a plain whole-file append on a hub note scrambled every project index before this rule existed. So never use `vault_append` / `obsidian_append_content` / a shell `>>` on a hub note.

Every route below does the same thing: address the `## <Section>` heading, append the index line to that heading's body. Pick the route matching your setup (`resolve-project.md` has the tool map). Let `<Section>` be `Sessions`, `Specs`, `PRDs`, `Transcripts`, `Prep`, `Research`, `Notes`, and so on.

### Route A: REST API built-in server (`vault_patch`)

The target is an **array of heading texts**, not a `::`-joined string. A bare string is rejected. The array is the full path from the top of the document down, and hub notes open with an H1 title, so the H1 is the first element. The `vault_get_document_map` call from the skill's gathering step supplies both the tree and the `version` token — the tree nests sections under the H1, e.g. `{"Vault Plugin": {"Sessions": {}, "Specs": {}}}`, which makes the path for `## Sessions` exactly `["Vault Plugin", "Sessions"]`.

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
- `ifMatch` makes the write fail with `ifMatch precondition failed` if the note changed since the document map, and nothing is written. Always pass it; it costs nothing extra.
- `rejectIfContentPreexists` makes a retry idempotent: if the line is already in the section, the call fails with `the target already contains the content to append` and nothing is duplicated. It combines with `within`.
- Copy heading keys **verbatim** from the document map. A repeated heading name carries a non-printable marker suffix; retyping it silently targets the wrong one.

**The two guards are the verification.** Between them, the only ways this call can go wrong — a stale note, a duplicate line — fail loudly and write nothing. A success response means the line is in its section and the neighbours are untouched, so go straight to the skill's confirm step rather than re-reading the hub.

### Route B: mcp-obsidian (`obsidian_patch_content`)

Here the heading path is a **string joined by `::`**, not an array. Hub notes begin with an H1 title, so `## Specs` is addressed as `<title>::Specs`; targeting bare `Specs` fails with `invalid-target` every time.

1. Take the hub note's first `# ` line, trimmed of the leading `# `, as `<title>`.
2. Call `obsidian_patch_content` with target type `heading`, target `<title>::<Section>`, operation `append`, and the index line as content.
3. **Verify** — no `ifMatch` or `rejectIfContentPreexists` here, so re-read the section and confirm the line is inside it and the neighbouring sections are untouched.

### Route C: no MCP

Read the note, insert the line at the end of the `## <Section>` section (immediately before the next `## ` heading, or at end-of-file), write the file back. Never a bare append. **Verify** as in Route B: re-read and confirm.

### If the section does not exist yet

Route A can pass `createTargetIfMissing: true`, but that places the heading without control over where. Prefer the explicit path on every route: read the hub note, add `## <Section>` after the last existing index section, insert the line under it, write back.
