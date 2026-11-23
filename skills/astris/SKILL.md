---
name: astris
description: Deploy and manage apps on Astris through the Astris MCP. Use when the user wants to check a repository with pre-flight before a deploy, deploy an app, read status, set variables, bind a hostname, restart, stop or start an app, explain a failed deploy, remove an app, create a Postgres or Redis database, back up a Postgres database, list a Postgres database's backups, delete a database, create a storage bucket, list or read this account's storage buckets, delete an empty storage bucket, list apps, databases, Compose stacks or projects, read a project, delete an empty project, read logs, get a Checkout or Customer Portal link, see what each plan gives, see the room left on the plan, or see which account and key it is acting as.
---

# Astris

Astris deploys and manages apps for the account that owns the API key.

## How to act

Act through the Astris MCP. If it is not installed, tell the user to install it using the MCP install steps in [install](references/install.md) and do nothing else meanwhile. The Astris CLI is published; its install line is in [install](references/install.md). Never call any host other than api.astris.run.

Read what the app needs before the first `deploy` call; see [Deploy flow](references/deploy-flow.md).

An app that serves no web page deploys the same way with `kind` `worker` or `kind` `cron`; the deploy flow says how.

If a create is refused because the account has no plan, do not retry it. Tell the user to choose a plan on the Billing page in the Console, then try again.
To get a Checkout link, call `billing` with action `checkout` and the plan the user chose, then give the user the link. Never choose a plan for the user.

Before a change to the data, call `database` with action `backup`. Redis has no backup.

Remove an app only when the user asks you to.
To remove it, call `app_remove` with its `app_id` and its exact name as `confirm_name`.
Removing the app deletes the app and its data. Its databases and storage buckets are separate and are not removed.
To stop an app without removing it, call `app_stop` with its `app_id`. To start it again, call `app_start`.
Delete an app, a database, a hostname, a bucket or a project only when the user asks you to.

These tools can delete or replace data: `env`, `domains`, `database`, `compose`, `rollback`, `app_remove`, `storage` and `project`. Your MCP client may ask the user before it runs one of them.

When a deploy fails or an app stops answering, call `diagnose` with its `app_id` and tell the user what the result says.

Before a deploy, call `usage` to see the room left.
Call `plans` to see what each plan gives, and `usage` for the room left.
To see which account and key you are acting as, call `whoami`.
Before the first deploy, call `whoami` and read `next_step.step`. If it is `choose_plan`, ask the user to choose a plan on the Billing page in the Console, then call `whoami` again before you deploy. If it is `create_app` or `none`, go ahead with the deploy. If it is any other value, show the user `next_step.sentence`.
To see the project names before a `deploy`, call `list` with `projects`.

## References

- [Tools](references/tools.md)
- [Deploy flow](references/deploy-flow.md)
- [Troubleshooting](references/troubleshooting.md)
- [Limits and refusals](references/limits-and-refusals.md)
- [Install](references/install.md)
- [Recipes for Node frameworks](references/recipes-node.md)
- [Python recipes](references/recipes-python.md)
- [Recipes: static sites, workers and cron jobs](references/recipes-static-workers-cron.md)
- [Go live](references/go-live.md)
