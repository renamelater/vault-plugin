---
name: setup
description: Set up the Obsidian second-brain vault structure these skills depend on.
disable-model-invocation: true
---

# Set Up the Second-Brain Vault

Bootstraps the folder structure and navigation doc these skills depend on, inside the user's own Obsidian vault.

## Step 1: Locate the Vault

- Ask: "Where is your Obsidian vault? (Give me the path — or tell me where to create a new one.)"
- If the path exists and contains a `.obsidian/` folder, it's a real vault — proceed.
- If the path exists but has no `.obsidian/` folder, confirm: "This folder isn't an Obsidian vault yet — set it up anyway? (Obsidian will adopt it when you open it as a vault.)"
- If creating fresh, create the folder.
- Everything below uses this path as `[vault]`.

## Step 2: Create the Structure

Create these folders and files, **skipping anything that already exists — never overwrite**:

```
[vault]/
├── context/
│   ├── about-me.md          ← stub: "# About Me" + prompt to fill in role, values, background
│   └── preferences.md       ← stub: "# Preferences" + prompt to fill in working/communication style
├── work/
│   ├── projects/
│   ├── general/
│   └── tasks/
│       ├── current.md       ← "# Work Tasks" heading only
│       └── archive.md       ← "# Archived Work Tasks" heading only
├── personal/
│   ├── projects/
│   ├── interests/
│   ├── general/
│   └── tasks/
│       ├── current.md       ← "# Personal Tasks" heading only
│       └── archive.md       ← "# Archived Personal Tasks" heading only
└── resources/
```

## Step 3: Write the Navigation Doc

- Copy `${CLAUDE_PLUGIN_ROOT}/skills/setup/template-vault-claude.md` to `[vault]/CLAUDE.md`.
- If `[vault]/CLAUDE.md` already exists, do NOT overwrite — show the user a diff summary and ask whether to merge the template's sections in or leave theirs alone.
- Personalize the `## Who I am` section: ask the user for a one-line description of themselves (role, what they work on) and insert it. If they'd rather skip, leave the placeholder.

## Step 4: Record the Vault Location

- Append this line to `~/.claude/CLAUDE.md` (create the file if missing):
  `Obsidian vault: [vault]`
- If a different `Obsidian vault:` line already exists there, ask which vault should be the default before changing it.
- This line is how every other vault skill finds the vault when run outside a linked project folder.

## Step 5: Optional — Obsidian MCP

Tell the user:
- The skills work out of the box with plain file access — no MCP required.
- If they want API-based access (Obsidian's search index, live vault state), offer to run `/vault:connect-obsidian`, which guides the full install-and-link flow. Only invoke it if they say yes.

## Step 6: Confirm

Report what was created vs. skipped (already existed), the vault path recorded, and suggest next steps:
- `/vault:new-project` to set up their first project
- `/vault:compress` at the end of any work session
