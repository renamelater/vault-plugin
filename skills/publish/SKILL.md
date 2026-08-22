---
name: publish
description: Publish a vault document to Notion. Use when the user says "publish", wants a vault file shared to Notion, or wants an already-published page refreshed after vault edits.
---

# Publish a Vault Document to Notion

When invoked, do the following:

Read `../../references/resolve-project.md` (relative to this skill's folder) before touching the vault: it owns the vault root and `[vault]` substitution, project resolution (**bucket** and **slug**), file placement, tool access, and no-match handling. Notion tool access and the no-tools branch: `../../references/notion.md` — read it now too.

## Step 1: Identify the Document

- The user names a vault file — that file is what gets published.
- Content that exists only in the conversation is saved into the vault first (placement rules in `resolve-project.md`), then published. The vault copy is the source of truth; the Notion page mirrors it.
- If no file is identifiable, ask which document to publish.

## Step 2: Read the Frontmatter

Open the file. A `notion:` field means this document already has a page — go to Step 5 (republish). No `notion:` field — continue to Step 3 (first publish).

## Step 3: Resolve the Destination (first publish)

The destination parent page is a property of the project, recorded in its hub note:

- Hub note has a `## Notion` section with a `Destination:` line → use it.
- Missing → ask the user where this project's published pages should live in Notion, offering a `notion-search` on the project name to surface candidates. Record the answer as `Destination: <url>` under `## Notion` in the hub note, following the insertion procedure in `../../references/vault-writes.md`.
- A destination named in the user's request wins for this call; the recorded line stays as it was.

## Step 4: Create the Page (first publish)

- Title: the document's H1. Body: the markdown after the H1, without the frontmatter block, ending with the footer (below).
- Create it under the destination parent. Continue at Step 6.

## Step 5: Republish

Fetch the page at `notion:` once, hash its content (drift-stamp procedure in `notion.md`), and compare against the file's `notion-hash:`:

- **Match** — clean: update the page in place with the current document body and a fresh footer. Continue at Step 6.
- **Mismatch** — **drift**: someone edited the page since the last publish. Stop and offer, in this order:
  1. Show what changed — compare the fetched page content to the vault document
  2. Fold their edits into the vault document, then republish
  3. Overwrite anyway
  4. Abort
  The fold-in path edits the vault file first, so the changes land in the source of truth, then updates the page from it.

## Step 6: Write Back the Frontmatter

After the create/update succeeds, fetch the page once and hash its content (drift-stamp procedure in `notion.md`). Record in the document's frontmatter (YAML quoting rules in `vault-writes.md`):

- `notion:` — the page URL
- `notion-published:` — today's date (real clock, per `vault-writes.md`)
- `notion-version:` — the footer's `<N>`
- `notion-hash:` — the hash of that post-publish fetch

Done when all four fields carry the values of the publish that just happened.

## Step 7: Confirm

Tell the user:
- The page URL and version
- First publish: the destination now recorded in the hub note
- Which drift option ran, if drift was hit

## Rich Content

Applies to every published body, first publish and republish alike:

- A URL on its own line publishes as-is — Notion renders it as a bookmark or embed, the right treatment for live prototypes, deploys, and artifacts.
- A local image reference (`![...](path)`) gets uploaded via the connector's file-upload tool and embedded — a vault path would be a dead link in Notion.
- Mermaid code blocks publish untouched — Notion renders them natively.

## The Footer

The last block of every published page:

```
---
v<N> · published from the vault <YYYY-MM-DD> · edits made directly to this page are folded in or overwritten at the next publish · previous versions: this page's history
```

`<N>` is `notion-version:` + 1 (first publish: 1). The date comes from the real clock — timestamp rules in `../../references/vault-writes.md`.

## Rules

- One document, one page — republish updates in place. A fresh page for an already-published document happens only when the user asks to move it (update `notion:` accordingly).
- The vault file is the source of truth — Notion-side edits survive by being folded into the vault.
