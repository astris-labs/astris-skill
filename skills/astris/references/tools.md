# Tools

| Tool | Description |
| --- | --- |
| `deploy` | Create an app from a public Git URL and queue its first deploy, or redeploy an existing app. Pass app_id to redeploy that app only. Other fields are ignored when app_id is set. Without app_id, project, name, git_url, and branch are required. |
| `status` | Read an application's status, hostnames, and at most five deployments. |
| `env` | This call replaces the whole set of variables, and omitted names are removed. No value is returned. |
| `domains` | Bind one astris.live hostname to an application. Omit slug to use the application slug. |
| `restart` | Restart an application. |
| `database` | Create a Postgres or Redis database or read its status. The password is returned only on create. |

Later tools take an Astris id from an earlier result.

## Arguments

| Tool | Argument | What the schema says |
| --- | --- | --- |
| `deploy` | `app_id` | Astris application id. When this is a non-empty string, redeploy that app and ignore every other field. |
| `deploy` | `project` | Project name. Required when app_id is absent. Created when this account has no project with that name. |
| `deploy` | `name` | App name, 1 to 63 characters. Required when app_id is absent. |
| `deploy` | `git_url` | Public http or https Git URL with no user or password. Required when app_id is absent. |
| `deploy` | `branch` | Git branch. Required when app_id is absent. |
| `deploy` | `build_path` | Path inside the repo. Default /. Used only when creating an app. |
| `deploy` | `dockerfile` | Dockerfile path inside the repo. Omit it for the default builder. Used only when creating an app. |
| `deploy` | `port` | Container port. Default 3000. Used only when creating an app. |
| `status` | `app_id` | Astris application id. |
| `env` | `app_id` | Astris application id. |
| `env` | `variables` | Flat object of strings. This object replaces the whole set. Omitted names are removed. |
| `domains` | `app_id` | Astris application id. |
| `domains` | `slug` | Hostname label. Omit it and the API uses the application slug. |
| `restart` | `app_id` | Astris application id. |
| `database` | `action` | create or status. |
| `database` | `project` | Project name. Required when action is create. |
| `database` | `name` | Database name. Required when action is create. |
| `database` | `kind` | postgres or redis. Required when action is create. |
| `database` | `database_id` | Astris database id. Required when action is status. |
