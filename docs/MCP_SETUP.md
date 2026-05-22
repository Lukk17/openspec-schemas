# MCP Setup

Human setup guide for the Model Context Protocol (MCP) servers shipped by this project. Covers prerequisites,
acquiring keys, setting environment variables system-wide, per-machine config files, and verification.

This file is for the **person** configuring the machine. The AI agent reads [AGENTS.md](../AGENTS.md) and uses
whatever MCP tools the running agent exposes; it doesn't need this document.

---

### What ships

Two committed templates in the repo root:

- [.mcp.json](../.mcp.json): Claude Code project scope. Schema key `mcpServers`. Env-var syntax:
  `${VAR}` and `${VAR:-default}`.
- [opencode.json](../opencode.json): OpenCode plus Kilo Code project scope (they share the file).
  Schema key `mcp` alongside `instructions`. Env-var syntax: `{env:VAR}` (no `$`).

Active servers in this project:

| Server     | Purpose                          | External dependency                           |
| ---------- | -------------------------------- | --------------------------------------------- |
| `context7` | Up-to-date library docs lookup   | none (hosted HTTP, free tier without API key) |

To disable: in `opencode.json` set `"enabled": false` on the block. In `.mcp.json` delete the block; Claude Code's
schema has no `enabled` flag.

---

### Step 1, install prerequisites

Context7 is an HTTP-based MCP, so there are no local binaries to install. Verify your agent can reach
`https://mcp.context7.com/mcp` from the machine you run it on. Corporate proxies and offline machines will need an
explicit allowlist or an alternative route.

---

### Step 2, acquire keys (optional)

Nothing is strictly required. Context7's free tier works without authentication; the `CONTEXT7_API_KEY` header is
forwarded empty by default and the upstream accepts the request.

| Key                | Where to get it                          | Needed when                         |
| ------------------ | ---------------------------------------- | ----------------------------------- |
| `CONTEXT7_API_KEY` | <https://context7.com/dashboard>         | you want paid tier or higher limits |

If you don't need the higher limits, leave the variable unset and the templates still parse and connect.

---

### Step 3, set environment variables system-wide

Prefer system-wide variables over per-shell so every agent inherits them: Claude Code, OpenCode, Kilo Code CLI,
JetBrains plugins, GUI apps.

#### Linux

Append to `/etc/environment` (system-wide, all users) or `~/.profile` (your user):

```bash
echo 'CONTEXT7_API_KEY=ctx7_xxxxxxxxxxxx' | sudo tee -a /etc/environment
```

Log out and back in to apply.

For interactive shells only, append to `~/.bashrc` or `~/.zshrc`:

```bash
export CONTEXT7_API_KEY=ctx7_xxxxxxxxxxxx
```

#### macOS

Append to `~/.zprofile` (loaded by Terminal and GUI sessions started from Terminal):

```bash
echo 'export CONTEXT7_API_KEY=ctx7_xxxxxxxxxxxx' >> ~/.zprofile
```

For GUI apps launched from Finder (Claude Desktop, IDEs), set via `launchctl`:

```bash
launchctl setenv CONTEXT7_API_KEY ctx7_xxxxxxxxxxxx
```

Persist `launchctl` values across reboots with a `~/Library/LaunchAgents/setenv.<name>.plist` entry or a tool like
[direnv](https://direnv.net/) scoped per project.

#### Windows

Set persistently for the current user (visible to new processes after restart):

```powershell
[Environment]::SetEnvironmentVariable('CONTEXT7_API_KEY', 'ctx7_xxxxxxxxxxxx', 'User')
```

System-wide (requires Administrator):

```powershell
[Environment]::SetEnvironmentVariable('CONTEXT7_API_KEY', 'ctx7_xxxxxxxxxxxx', 'Machine')
```

Verify in a fresh PowerShell session:

```powershell
$env:CONTEXT7_API_KEY
```

Restart your terminal, IDE, and any running agent shell after changing system variables. Claude Desktop must be fully
quit (system-tray icon) and relaunched.

---

### Step 4, variables you can override

Every variable in the table is optional. The defaults are baked into the template files, either via `${VAR:-default}`
in [.mcp.json](../.mcp.json) or as hardcoded literals in [opencode.json](../opencode.json).
Set a variable only when you need to supply a real token.

| Variable            | Used by    | Effect when unset                       |
| ------------------- | ---------- | --------------------------------------- |
| `CONTEXT7_API_KEY`  | `context7` | header sent empty (Context7 free tier)  |

Important asymmetry between the two files:

- [.mcp.json](../.mcp.json) (Claude Code) uses `${VAR:-default}`, so every env var has a resolvable
  fallback. Files parse with no env set.
- [opencode.json](../opencode.json) (OpenCode plus Kilo) uses `{env:VAR}` for tokens. OpenCode's substitution syntax
  has no `:-default` fallback; an unset variable resolves to the empty string.

---

### Step 5, user-global configs (per machine, not committed)

The committed templates cover project scope. Three more files run user-global; none of them ships in the import.

#### Claude Code user-global

Path: `~/.claude.json`. Same `mcpServers` schema as the project file. Same `${VAR}` substitution.

Used when you're working outside any project that defines its own `.mcp.json`. Project file wins on name collisions.

If the file already exists, merge the `mcpServers` block by hand; Claude Code keeps other top-level keys there too
(`theme`, `editorMode`, etc.).

To populate from scratch:

```bash
cp .mcp.json ~/.claude.json
```

#### Claude Desktop

Paths:

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: no official Claude Desktop build.

Same `mcpServers` schema. **Does not support env-var substitution.** Replace each `${VAR}` with the actual value. This
file is per-machine personal, so inlining is acceptable; just never commit it.

Example:

```json
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "ctx7_actual_value_here"
      }
    }
  }
}
```

#### Codex CLI

Path: `~/.codex/config.toml`. Different format (TOML, not JSON). Global-only, no project scope.

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp@latest"]
```

Codex has no built-in env-var substitution. If you want secrets out of the file, wrap the command in a small shell
script that reads from your secrets manager.

---

### Env-var substitution support matrix

Useful when debugging why a variable isn't being picked up.

| Tool / config file                              | Substitution syntax            | Supported fields                              |
| ----------------------------------------------- | ------------------------------ | --------------------------------------------- |
| Claude Code, `.mcp.json` and `~/.claude.json`   | `${VAR}`, `${VAR:-default}`    | `command`, `args`, `env`, `url`, `headers`    |
| OpenCode / Kilo Code, `opencode.json`           | `{env:VAR}`                    | string values in any MCP field                |
| Claude Desktop, `claude_desktop_config.json`    | **not supported**              | inline literal values                         |
| Codex CLI, `~/.codex/config.toml`               | TOML; no built-in substitution | inline literal values                         |

---

### Verifying the setup

After editing the configs and exporting env vars, restart the agent. Then:

- **Claude Code**: run `claude mcp list` to see which servers loaded.
- **OpenCode / Kilo Code**: start the agent and check the startup log for MCP connection lines, or run `/mcp` inside
  the shell.
- **Claude Desktop**: open the Developer panel; the MCP indicator in the input box shows connected servers.
- **Codex CLI**: start `codex` and check `/mcp` (Codex 0.20+).

If a server fails to connect:

- Claude Code logs the reason to its standard log.
- Claude Desktop logs land in `~/Library/Logs/Claude/` (macOS) or `%APPDATA%\Claude\logs\` (Windows). Files named
  `mcp-server-<name>.log` carry the per-server stderr.

---

### Adding or changing a server

Edit [.mcp.json](../.mcp.json) and [opencode.json](../opencode.json) directly. Both files are project-scoped; any
addition becomes available to every contributor on their next agent restart. Document the new server's purpose,
required env vars, and external dependency in the **What ships** table above so the next person setting up the
machine knows what they're agreeing to.
