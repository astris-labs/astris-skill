---
name: astris
description: Deploy and manage apps on Astris through the Astris MCP. Use when the user wants to deploy an app, read status, set variables, bind a hostname, restart an app, or create a Postgres or Redis database.
---

# Astris

Astris deploys and manages apps for the account that owns the API key.

## How to act

Act through the Astris MCP. If it is not installed, tell the user to install it using the MCP install steps in [install](references/install.md) and do nothing else meanwhile. The CLI is not yet available to builders. Never call any host other than api.astris.run.

Read what the app needs before the first `deploy` call; see [Deploy flow](references/deploy-flow.md).

## References

- [Tools](references/tools.md)
- [Deploy flow](references/deploy-flow.md)
- [Limits and refusals](references/limits-and-refusals.md)
- [Install](references/install.md)
