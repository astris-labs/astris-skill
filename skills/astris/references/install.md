# Install

## Astris MCP

Astris MCP is a remote server at `https://api.astris.run/mcp`. Your client sends your Astris API key as `Authorization: Bearer` with the key. Claude Code and Codex read the key from the `ASTRIS_API_KEY` environment variable, so the key stays out of the commands you type, where your shell's history would keep it. In Cursor's file, replace `PASTE_KEY_HERE` with the key. Do not commit a file that contains the key.

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

Set `ASTRIS_API_KEY` to your key, then run this command. The command names that variable. It does not contain the key. To set it without putting the key in your shell's history, run `read -rs ASTRIS_API_KEY && export ASTRIS_API_KEY` on macOS or Linux, or `$env:ASTRIS_API_KEY = [System.Net.NetworkCredential]::new('', (Read-Host -AsSecureString 'Astris API key')).Password` in Windows PowerShell, then paste the key and press Enter. Start Claude Code from that terminal.

```text
claude mcp add --transport http astris https://api.astris.run/mcp --header 'Authorization: Bearer ${ASTRIS_API_KEY}'
```

### Codex

```toml
[mcp_servers.astris]
url = "https://api.astris.run/mcp"
bearer_token_env_var = "ASTRIS_API_KEY"
tool_timeout_sec = 180
```

Set `ASTRIS_API_KEY` to the key. The file names that variable. It does not contain the key.

## Command line

Needs Node 22 or later.

```text
npm install --global @astris-labs/cli
```

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
