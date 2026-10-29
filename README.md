# fleet-install

Public host for the fleet MCP installer. One command registers a `sandraschi` MCP
server into every AI client detected on the machine.

```powershell
irm https://raw.githubusercontent.com/sandraschi/<repo>/<branch>/install.ps1 | iex
```

Each repo carries a thin `install.ps1`; the real logic lives here so there is one
copy to maintain rather than ~130 drifting copies.

## What it does

- Detects Claude Desktop, Cursor, Antigravity, Windsurf, OpenCode and Claude Code.
- Backs up each client config, **merges one entry** (never replaces the file).
- Resolves the server's stdio launch command from the repo's `mcpb/manifest.json`
  (`server.mcp_config`) first, then `[project.scripts]`, then
  `src/<pkg>/server.py`, then `run_server.py`.
- **Proves each tier** with a real `initialize` handshake over stdio before
  trusting it, so a stale manifest falls through instead of breaking a client.
- Reuses the client alias that already points at the repo (so `reversing-mcp`
  stays `revops`), and fails closed if the alias belongs to a different server.
- Never re-syncs an existing venv. `uv sync` prunes optional extras
  (`mcp-central-docs` BUG-091); it only builds when `.venv` is absent, using
  `SyncExtras` parsed from `fleet-start.config.ps1`.

## What it does NOT do

- It does **not** build or serve the webapp. That is `start.ps1` (fleet launcher:
  ports from `fleet-start.config.ps1`, uvicorn + vite).
- It does **not** download or run a Tauri/NSIS desktop installer. Those live as
  GitHub release assets; run them deliberately, not as a side effect of
  registering an MCP server.

## Flags

```
-Check      diagnose only, writes nothing
-DryRun     show per-client changes without writing
-Uninstall  remove only the entry this installer owns
-List       show which clients already have the entry
-Json       machine-readable result
-Yes         allow unattended prerequisite install (uv via winget)
```

## Source of truth

Canonical copies live in `mcp-central-docs/scripts/` (private). These files are
the public mirror; drift is checked the same way `install-mcp-clients.ps1` is
checked in NSIS installers.
