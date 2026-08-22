---
name: connect-obsidian
description: Connect Obsidian's Local REST API to Claude Code as an MCP server. Use when the user wants the Obsidian MCP, or asks to install or link the Local REST API.
---

# Connect Obsidian to Claude Code

Guides the user through installing Obsidian's Local REST API plugin and wiring it to Claude Code as an MCP server. Remind the user up front: this is **optional** — the vault skills work with plain file access; the MCP adds Obsidian's own search index and API access.

## Step 1: Check What's Already There

Run these checks before asking the user to do anything:

- **REST API reachable?** `curl -sk -m 5 https://127.0.0.1:27124/` — a JSON reply means the plugin is installed and running (note the `versions.self` value and whether `authenticated` is true). Also try `http://127.0.0.1:27123/` (the insecure port, if enabled).
- **MCP already configured?** `claude mcp list` — look for an entry pointing at mcp-obsidian or port 27124. If one exists, run `claude mcp get <name>` and read its `Scope:` line. `User config` is correct. `Local config` means it was registered in this folder only: re-add it at user scope in Step 4 (same command, the `-s user` flag), then continue to Step 5.
- **Usage rule in place?** `~/.claude/CLAUDE.md` has an `## Obsidian MCP` section (written by Step 5).

If all three are in place, tell the user they're already set up and stop. If only the rule is missing, jump to Step 5.

## Step 2: Install the Local REST API Plugin (user does this in Obsidian)

Skip if Step 1 found the API reachable.

- Open the plugin page for them: `open "obsidian://show-plugin?id=obsidian-local-rest-api"` (deep link — opens Obsidian at the right screen; on Linux use `xdg-open`)
- If the deep link doesn't work, tell them: Obsidian → Settings → Community plugins → (turn off Restricted mode if prompted) → Browse → search "Local REST API" (by Adam Coddington) → Install → Enable
- Wait for them to confirm, then re-run the reachability check from Step 1.

## Step 3: Get the API Key

- Tell the user: Obsidian → Settings → Local REST API → copy the **API Key** shown there, and paste it here.
- Verify it immediately:
  `curl -sk -m 5 -H "Authorization: Bearer <key>" https://127.0.0.1:27124/`
  — success is `"authenticated": true` in the reply.
- If verification fails, diagnose in this order: Obsidian not running → the plugin disabled → a different port configured in the plugin settings (ask the user to read the port from the settings screen) → a mistyped key.

## Step 4: Register the MCP Server

Both routes register at **user scope** (`-s user`): the vault is one per user, so the server belongs to every project. The default scope, `local`, would register it in the current folder only, and every other project would run without it.

Two routes — pick by what's available:

### Route A — the plugin's built-in MCP server (preferred: official, zero extra installs)

This is the integration the plugin's own README documents for Claude Code. Local REST API v5+ (and late 4.x) serves MCP itself, running inside Obsidian — so it also sees live vault metadata, the active file, and the command palette, which an external bridge cannot. Probe it first:

```
curl -sk -m 5 -X POST https://127.0.0.1:27124/mcp/ -H "Authorization: Bearer <key>" -H "Content-Type: application/json" -H "Accept: application/json, text/event-stream" -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"probe","version":"1.0"}}}'
```

If it answers with an MCP `initialize` result, register it (note the trailing slash on `/mcp/`):

```
claude mcp add -s user --transport http obsidian https://127.0.0.1:27124/mcp/ --header "Authorization: Bearer <key>"
```

If `claude mcp list` then fails on TLS (the API uses a self-signed certificate), two documented fixes, in order of preference:
1. Trust the plugin's certificate — download it from `https://127.0.0.1:27124/obsidian-local-rest-api.crt` (e.g. to `~/.claude/obsidian-local-rest-api.crt`) and point Node at it with `NODE_EXTRA_CA_CERTS=<path>` in Claude Code's `settings.json` `env`. The plugin regenerates its certificate roughly yearly — when the connection starts failing on TLS again, re-download the cert to the same path.
2. Enable "Enable Insecure HTTP Server" in the plugin settings and use `http://127.0.0.1:27123/mcp/` instead. Localhost-only, so the exposure is limited — but say so.

Note the built-in server's tool names differ from mcp-obsidian's `obsidian_*` names; the vault skills tolerate this (they fall back by function).

### Route B — mcp-obsidian bridge (alternative: exact `obsidian_*` tool-name parity)

Use when the plugin is older than the built-in MCP server, or when exact tool-name parity with these skills matters. Requires `uv` (check with `uvx --version`; if missing, it's `brew install uv` on macOS, or see https://docs.astral.sh/uv/getting-started/installation/).

```
claude mcp add -s user obsidian -e OBSIDIAN_API_KEY=<key> -e OBSIDIAN_HOST=127.0.0.1 -e OBSIDIAN_PORT=27124 -- uvx --with "mcp<2" mcp-obsidian
```

The `--with "mcp<2"` pin matters: as of mid-2026, mcp-obsidian 0.2.x crashes on startup with MCP SDK 2.0 (`'Server' object has no attribute 'list_tools'`). If a future mcp-obsidian release supports SDK 2.0, the pin can be dropped.

### Version note

Whatever route: if Step 1 found a plugin version below 4.1.3, tell the user to update it first — 4.1.3 patched an authenticated path-traversal vulnerability (GHSA-62gx-5q78-wrvx). The minimum Obsidian version moves between plugin releases (5.0.2 and 5.0.3 accept 1.8.7, 5.1.0 needs 1.13.1): read it from the Step 1 reply, which reports both `versions.obsidian` and `manifest.minAppVersion`, rather than from a number written here.

## Step 5: Verify and Write the Usage Rule

- `claude mcp list` — the `obsidian` entry should show **Connected**.
- Write the usage rule. A connected server is not enough: nothing tells Claude to prefer it over shell reads of vault files, and in sessions that favour shell tools it goes unused. Append this section to `~/.claude/CLAUDE.md` (create the file if missing; if an `## Obsidian MCP` section already exists, replace it). Route A text:

```
## Obsidian MCP
The `obsidian` MCP server (Obsidian Local REST API built-in server, user scope) is connected on this machine. Its tools are `vault_read`, `vault_get_document_map`, `vault_patch`, `vault_append`, `vault_list`, `search_simple`, `search_query`. Use it for every read and write of a file under the Obsidian vault, including the session-open reads a project CLAUDE.md asks for. This overrides any general preference for shell tools: never `cat`, `sed`, or `>>` a vault file while the MCP is connected. If the tool schemas are deferred, load them with ToolSearch first. Paths are relative to the vault root, e.g. `work/projects/my-project/my-project.md`.
```

  For Route B, name the server as "mcp-obsidian bridge" and list its tools instead: `obsidian_get_file_contents`, `obsidian_patch_content`, `obsidian_append_content`, `obsidian_list_files_in_vault`, `obsidian_list_files_in_dir`, `obsidian_simple_search`, `obsidian_complex_search`.
- Remind the user: the MCP's tools and the new rule appear in **new** sessions, not the current one.

## Step 6: Confirm

Report: plugin version found, which route was configured, the scope, the health-check result, and that the usage rule was written to `~/.claude/CLAUDE.md`. If anything was left unfinished (e.g. user needs to install uv), state exactly what remains.
