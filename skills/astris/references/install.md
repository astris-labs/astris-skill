# Install

## Astris MCP

Astris MCP is a remote server at `https://api.astris.run/mcp`. Your client sends your Astris API key as `Authorization: Bearer PASTE_KEY_HERE`. Replace `PASTE_KEY_HERE` with the key. Do not commit a file that contains the key.

### Cursor

Paste this into `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "astris": {
      "url": "https://api.astris.run/mcp",
      "headers": {
        "Authorization": "Bearer PASTE_KEY_HERE"
      }
    }
  }
}
```

### Claude Code

```text
claude mcp add --transport http astris https://api.astris.run/mcp --header "Authorization: Bearer PASTE_KEY_HERE"
```

### Codex

```toml
[mcp_servers.astris]
url = "https://api.astris.run/mcp"
bearer_token_env_var = "ASTRIS_API_KEY"
tool_timeout_sec = 180
```

Set `ASTRIS_API_KEY` to the key. The file names that variable. It does not contain the key.

## This skill

### Claude Code

```text
claude plugin marketplace add astris-labs/astris-skill
claude plugin install astris@astris
```

In a session, run `/reload-plugins`.

### Cursor

Import the repository `astris-labs/astris-skill` in Customize with From GitHub Repository. Then install the plugin.

### Codex

In Codex, type:

```text
$skill-installer https://github.com/astris-labs/astris-skill/tree/main/skills/astris
```
