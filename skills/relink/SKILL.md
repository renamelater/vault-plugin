---
name: relink
description: Point the current code folder at an existing vault project.
disable-model-invocation: true
---

# Relink Project to Vault

Use this skill when the code folder name and the vault project slug don't match (e.g. cwd is `personal site_mw` but the vault project is `portfolio-site-mw`), or when an existing code folder needs a `CLAUDE.md` pointing at a vault project that already exists.

This skill **does not create a new vault project**. If no vault project exists yet, tell the user to run `/vault:new-project` first.

When invoked, do the following:

Vault root, `[vault]` substitution, and file-access rules: `../../references/resolve-project.md` — read it before touching the vault.

## Step 1: Confirm the Code Folder
- Read the current working directory (`cwd`)
- Tell the user: "Linking the code folder at `[cwd]` — is that right?"
- Wait for confirmation before continuing

**Guardrails — before continuing, check the cwd:**
- If cwd is `[vault]` or anywhere inside it → STOP. Tell the user: "You're in the Obsidian Vault, not a code folder. Open your code folder in Cursor and run /vault:relink from there."
- If cwd is the home directory → STOP with the same warning.

## Step 2: List Existing Vault Projects
- Use `obsidian_list_files_in_vault` to list folders under `work/projects/` and `personal/projects/`
- Show the user the list and ask: "Which vault project should this folder link to?"
- They'll respond with the slug (e.g. `portfolio-site-mw`) and optionally `work` or `personal` if it's ambiguous

## Step 3: Verify the Match and Find the Hub Note
- The project **hub note** is `[work or personal]/projects/[chosen-slug]/[chosen-slug].md`
- If that file doesn't exist, fall back to `[work or personal]/projects/[chosen-slug]/README.md` (older projects)
- If neither exists, tell the user: "I don't see `[chosen-slug]` in the vault. Run `/vault:new-project` to create it, then re-run `/vault:relink`." Stop.
- Use the hub note filename you actually found in the CLAUDE.md `Overview:` line below

## Step 4: Check for Existing CLAUDE.md
- Check if `./CLAUDE.md` already exists in cwd
- If it does, read it and tell the user: "There's already a CLAUDE.md here pointing at `[whatever vault path it has]`. Rewrite its vault sections to point at `[chosen-slug]`?"
- Wait for confirmation. If they decline, stop.
- **Preserve foreign sections.** When rewriting, replace only the sections the template owns (listed in `../../references/template-project-claude.md`) and keep every other section intact, in place.

## Step 5: Write CLAUDE.md to the Current Working Directory

Write `./CLAUDE.md` (the cwd) from the block in `../../references/template-project-claude.md`, substituting its placeholders: `[bucket]` and `[slug]` from Step 2, `[hub-note-filename]` and `[Project Name]` from the hub note found in Step 3, `[one line description]` from the hub note's `## What is this` section if present, `[cwd]` is the current working directory.

## Step 6: Update the Hub Note with the Code Folder Path

**Read-modify-write — never blind-append.**

- Read the hub note
- If a `## Code` section already exists, replace its content with the cwd path
- If it doesn't exist, insert a new `## Code` section after `## Started` (NOT at the end of the file — appending to the file would land it after the index sections)
- Write the modified file back

This makes the link bidirectional — the code folder knows where the vault is (via `CLAUDE.md`), and the vault knows where the code is (via the hub note's `## Code` section).

## Step 7: Confirm
Tell the user:
- CLAUDE.md written at: `[cwd]/CLAUDE.md` → pointing at `[work or personal]/projects/[chosen-slug]/`
- Hub note updated at: `[its path]` → `## Code` set to `[cwd]`
- `/vault:compress` from this folder will now find the vault project even though the folder names differ

## Rules
- Never invoke `/vault:new-project` from this skill — if the vault project doesn't exist, tell the user to run it themselves
- Never modify the cwd folder name or the vault slug — this skill only writes link files
- Both writes (CLAUDE.md and hub note) must succeed; if one fails, tell the user which one
