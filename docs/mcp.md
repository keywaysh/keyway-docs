---
sidebar_position: 5
title: MCP Server
---

# MCP Server

The Keyway MCP server allows AI assistants to securely access your secrets.

## Prerequisites

```bash npm2yarn
npm install -g @keywaysh/cli
```

```bash
keyway login
```

---

## Configuration

### Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "keyway": {
      "command": "npx",
      "args": ["-y", "@keywaysh/mcp"],
      "cwd": "/path/to/your/project"
    }
  }
}
```

### VS Code / Cursor

Add to settings.json (`Cmd+Shift+P` → "Preferences: Open User Settings (JSON)"):

```json
{
  "mcp.servers": {
    "keyway": {
      "command": "npx",
      "args": ["-y", "@keywaysh/mcp"],
      "cwd": "${workspaceFolder}"
    }
  }
}
```

### Claude Code CLI

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "keyway": {
      "command": "npx",
      "args": ["-y", "@keywaysh/mcp"]
    }
  }
}
```

### Windsurf

Add to Windsurf MCP settings:

```json
{
  "mcpServers": {
    "keyway": {
      "command": "npx",
      "args": ["-y", "@keywaysh/mcp"],
      "cwd": "/path/to/your/project"
    }
  }
}
```

---

## Available Tools

### keyway_list_secrets

List secret names (without values).

```json
{ "environment": "production" }
```

### keyway_get_secret

Get a specific secret value.

```json
{
  "name": "DATABASE_URL",
  "environment": "production"
}
```

### keyway_set_secret

Create or update a secret.

```json
{
  "name": "API_KEY",
  "value": "sk-...",
  "environment": "production"
}
```

### keyway_inject_run

Run a command with secrets as env vars.

```json
{
  "command": "npm",
  "args": ["run", "dev"],
  "environment": "development",
  "timeout": 300000
}
```

### keyway_list_environments

List available environments.

```json
{}
```

---

## Example Prompts

**"What secrets are in production?"**
→ Uses `keyway_list_secrets`

**"Get the DATABASE_URL for staging"**
→ Uses `keyway_get_secret`

**"Run the tests with the development secrets"**
→ Uses `keyway_inject_run`

**"Add a new API_KEY secret with value xyz"**
→ Uses `keyway_set_secret`

---

## Security

- **Token reuse** - Uses CLI's encrypted token (`~/.keyway/.key`)
- **No logging** - Secret values never logged
- **Output masking** - `inject_run` masks secrets in output
- **Shell injection prevention** - Commands run with `shell: false`
- **Audit trail** - All accesses logged (viewable in dashboard)

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `KEYWAY_API_URL` | Override API URL |

---

## Troubleshooting

**"Not logged in"** → Run `keyway login`

**"No vault found"** → Ensure `cwd` points to a git repo with GitHub remote

**"Permission denied"** → Check GitHub repo access
