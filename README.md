# Astris Skill

This skill tells a coding agent how to deploy and manage apps on Astris through the Astris MCP.

Ask it to deploy an app, read status, set variables, bind a hostname, restart an app, or create a Postgres or Redis database.

## Claude Code

Run these two commands, then run `/reload-plugins` in a session.

```text
claude plugin marketplace add astris-labs/astris-skill
claude plugin install astris@astris
```

## Cursor

Import the repository `astris-labs/astris-skill` in Customize with From GitHub Repository. Then install the plugin.

## Codex

In Codex, type:

```text
$skill-installer https://github.com/astris-labs/astris-skill/tree/main/skills/astris
```

The same commands, and the Astris MCP setup, are in `skills/astris/references/install.md`.
