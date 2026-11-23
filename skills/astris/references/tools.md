# Tools

| Tool | Description |
| --- | --- |
| `preflight` | Check the repository files you send, before you create an app. Returns a verdict of pass, fixable or refuse, and one row per check with its name, its result (pass, fixable, refuse or did not run) and its reason. |
| `deploy` | Create an app from a Git URL and queue its first deploy, or redeploy an existing app. Pass app_id to redeploy that app only. Other fields are ignored when app_id is set, except wait. Pass wait true to wait for the deployment this call queues to end, then return its outcome: done, error, cancelled, or still running, check status. wait is not ignored when app_id is set, and it does not apply to an SSH git_url. Without app_id, project, name, git_url, and branch are required. Call list with resource projects first and pass the exact name of an existing project; a name that matches none creates a new project. An SSH git_url creates the app and returns its deploy key without deploying. Add that key to the repository as a read-only deploy key, then deploy again with the returned application_id as app_id. Omit build for the default Nixpacks builder (no Dockerfile needed). Pass build "dockerfile" with the name of a Dockerfile in the top folder of the repository for custom builds. Every app builds from the top folder of its repository. Pass build "static" for HTML/CSS/JS-only sites served on port 80; port is ignored for static sites. Pass kind worker with a command to run it as the app's process with no public address, or kind cron with a command and a five-field schedule to run it on a timer. A cron app must keep running: it keeps its normal start, and the schedule runs the command inside it. |
| `status` | Read an application's status, hostnames, and at most five deployments. |
| `env` | This call replaces the whole set of variables, and omitted names are removed. No value is returned. |
| `env_names` | Read the names of an application's environment variables. No value is returned. This tool changes nothing; env is the call that replaces the set. |
| `domains` | Bind a hostname to an application, verify a custom hostname once its DNS is set, read one hostname, or remove a hostname from an application. Without action, or with action bind, pass slug for an astris.live name and omit it to use the application slug, or pass hostname for the builder's own domain. With action verify, pass the domain_id from the bind result. A custom bind returns the records to create and routes nothing until verify reports status active. With action status, pass the domain_id; it reads the domain and its certificate word and changes nothing. With action remove, pass the domain_id and confirm true; the hostname stops routing and the record is gone. |
| `deploy_hook` | Read this app's push-to-deploy URL. The URL is a secret, and anyone who holds it can start a deploy of this app. Never paste it into a public place, a log, a file in the repository, a commit message or an issue. To wire up GitHub, open the repository, then Settings, Webhooks, Add webhook, put the URL in Payload URL, set Content type to application/json, and subscribe to push events. |
| `restart` | Restart an application. |
| `rollback` | Roll an application back to its last working deploy, or to a chosen commit from its last ten deploys. Env values return to that deploy's values until the next deploy. |
| `database` | Create a Postgres or Redis database, read its status, run a backup of a Postgres database, list its backups, restore a Postgres database from one of its backups, import an uploaded dump, or delete one. The password is returned only on create. |
| `compose` | Compose stacks cannot be created or redeployed. Deploy each service from your compose file as an app. For a database service, create a managed database instead. Stop, start, read the status of, or remove an existing stack by its compose_id. Removing a stack deletes its volumes and all their data. |
| `diagnose` | Explains a failed deploy or an app that stopped (its exit code, or that it went over its memory limit), and returns the raw evidence when it cannot tell the cause. An empty diagnosis comes with a note that says what Astris checked. |
| `list` | List the apps, the databases, the Compose stacks, or the projects on this account. |
| `logs` | Read the runtime or deploy log lines for an application. |
| `usage` | Read this account's plan, its pools, and the RAM, CPU, and disk it has used. |
| `plans` | Read the RAM, CPU, and disk each plan gives. |
| `whoami` | Read which account and which API key this request uses. |
| `app_remove` | Remove an app and its data. It runs only when confirm_name is the app's exact name. Its databases and storage buckets are not removed. |
| `app_start` | Start an application. |
| `app_stop` | Stop an application. |
| `storage` | Create a storage bucket and get its S3 credentials, list this account's buckets, read one bucket, issue a new key for one bucket, or delete an empty bucket. The secret key appears only when you create a bucket or issue a new key, so store it now. |
| `billing` | Get a Checkout link for a plan, or a Customer Portal link for this account, or change the plan of an account that already pays. Checkout needs a plan name. The API says which plans are on sale. Portal takes no plan. Switch needs a plan name. A smaller plan starts at the next renewal. A bigger plan starts now and charges the card on file, so the first switch call changes nothing and answers what it costs and a confirm value: show that to the person, wait for their yes, then call switch again with the same plan and that confirm value. Never send confirm without that yes. With a deploy key, that second call mails the account owner, who approves on the Console Billing page; the plan changes only then. Keep cancels a smaller plan that waits for the renewal. |
| `project` | Read one project on this account, or delete one that holds nothing. Deleting a project is final, and a project that still holds an application, a database, or a compose stack is refused by the route. |

Later tools take an Astris id from an earlier result.

## Arguments

| Tool | Argument | What the schema says |
| --- | --- | --- |
| `preflight` | `files` | Array of files, each with a repository-relative path using forward slashes and the whole file contents as text. Send each path once. |
| `preflight` | `env_names` | Optional. The app's variable names, as `env_names` returns them. Names only, never a value. |
| `deploy` | `app_id` | Astris application id. When this is a non-empty string, redeploy that app and ignore every other field except wait. |
| `deploy` | `project` | Project name. Required when app_id is absent. Created when this account has no project with that name. |
| `deploy` | `name` | App name, 1 to 63 characters. Required when app_id is absent. |
| `deploy` | `git_url` | Git URL with no user or password: an https URL for a public repository, or git@host:owner/repo.git or ssh://git@host/owner/repo.git for a private one. Required when app_id is absent. |
| `deploy` | `branch` | Git branch. Required when app_id is absent. |
| `deploy` | `build_path` | Only / is accepted, because every app builds from the top folder of its repository. Omit it. Used only when creating an app. |
| `deploy` | `dockerfile` | Name of a Dockerfile in the top folder of the repository, such as Dockerfile or Dockerfile.prod, with no folder. Omit it for the default builder. Used only when creating an app. |
| `deploy` | `build` | Build kind: nixpacks (default), dockerfile, or static. Omit it for nixpacks. Used only when creating an app. |
| `deploy` | `publish_directory` | Publish directory for static sites. Used only with build "static". |
| `deploy` | `spa` | Single-page app mode for static sites. Used only with build "static". |
| `deploy` | `port` | Container port. Default 3000. Used only when creating an app. |
| `deploy` | `kind` | App kind: web (default), worker, or cron. Used only when creating an app. |
| `deploy` | `command` | Command a worker runs as its process, or a cron app runs on its schedule. Each cron run is stopped after 600 seconds, and only the first 1 MiB of its output is kept. 1 to 512 characters, no newline. Used only when creating an app. |
| `deploy` | `schedule` | Cron schedule, five space-separated fields, for example 0 3 * * *. Used only when creating an app. |
| `deploy` | `wait` | Wait for the deployment this call queues to end, then return its outcome: done, error, cancelled, or still running, check status. Not ignored when app_id is set. Not used with an SSH git_url. |
| `status` | `app_id` | Astris application id. |
| `env` | `app_id` | Astris application id. |
| `env` | `variables` | Flat object of strings. This object replaces the whole set. Omitted names are removed. |
| `env_names` | `app_id` | Astris application id. |
| `domains` | `app_id` | Astris application id. |
| `domains` | `hostname` | The builder's own hostname, for a custom domain. Takes precedence over slug. |
| `domains` | `slug` | Hostname label. Omit it and the API uses the application slug. |
| `domains` | `action` | bind (the default), verify, remove, or status. |
| `domains` | `domain_id` | Astris domain id from a bind result. Required when action is verify, remove, or status. |
| `domains` | `confirm` | Set to true to go ahead with remove. The remove sends nothing without it. |
| `deploy_hook` | `app_id` | Astris application id. |
| `restart` | `app_id` | Astris application id. |
| `rollback` | `app_id` | Astris application id. |
| `rollback` | `commit` | Optional. 7 to 40 hex characters of a commit from the last ten deploys. |
| `database` | `action` | create, status, backup, backups, restore, import, or delete. |
| `database` | `project` | Project name. Required when action is create. |
| `database` | `name` | Database name. Required when action is create. |
| `database` | `kind` | postgres or redis. Required when action is create. |
| `database` | `database_id` | Astris database id. Required when action is status, backup, backups, restore, import, or delete. |
| `database` | `dump_key` | Backup key from action backups. Required when action is restore. |
| `database` | `upload_id` | Upload grant id from POST /v1/databases/:id/import/upload. Required when action is import. |
| `database` | `confirm` | Set to true to go ahead with delete, restore, or import. Without it, the call does nothing. |
| `compose` | `action` | deploy, stop, start, status, or remove. Defaults to deploy. |
| `compose` | `compose_id` | Astris compose stack id. Required for stop, start, status, or remove. |
| `compose` | `confirm` | The stack's exact name. Required when action is remove. |
| `diagnose` | `app_id` | Astris application id. |
| `list` | `resource` | apps, databases, compose, or projects. |
| `logs` | `app_id` | Astris application id. |
| `logs` | `source` | runtime or deploy. Omit it for runtime. |
| `logs` | `tail` | How many lines. At most 500. |
| `logs` | `deployment` | Which of the five newest deployments to read, 1 is the newest. Only with source deploy. |
| `app_remove` | `app_id` | Astris application id. |
| `app_remove` | `confirm_name` | The app's exact name. Removal runs only when this matches. |
| `app_start` | `app_id` | Astris application id. |
| `app_stop` | `app_id` | Astris application id. |
| `storage` | `action` | create, list, show, rotate, or delete. |
| `storage` | `name` | Bucket name, 1 to 50 characters. Required when action is create. |
| `storage` | `bucket_id` | Astris bucket id. Required when action is show, rotate, or delete. |
| `billing` | `action` | checkout, portal, switch or keep. |
| `billing` | `plan` | Plan name for checkout or switch. One of starter, growth, pro, scale, business. The API says which of these are on sale. |
| `billing` | `confirm` | For switch to a bigger plan only: the confirm value from the first switch answer, sent after the person said yes. |
| `project` | `action` | show or delete. |
| `project` | `project_id` | Astris project id. Required when action is show or delete. |
| `project` | `confirm` | Set to true to go ahead with delete. The delete sends nothing without it. Ask the user, then call again with confirm true. Other actions ignore it. |

`usage` takes no arguments.
`plan` can be null. That is a normal result, not an error. When `plan` is null, `pools`, `ram_remaining_mb`, `cpu_remaining_milli` and `disk_remaining_gb` are null too.
Pool RAM is `ram_mb`, in MiB. Pool CPU is `cpu_milli`. Pool disk is `disk_gb`.
`used.ram_mb` counts apps, databases and Compose stacks. `used.cpu_milli` and `used.disk_gb` count apps only.
`plans` takes no arguments.
The five plans, in ladder order, are starter, growth, pro, scale, and business.
Each plan has `ram_mb` in MiB, `cpu_milli` in mCPU, and `disk_gb` in GB.
`whoami` takes no arguments.
`whoami` returns the account `id`, `email`, `status`, `plan`, and `created_at`; the key `id`, `name`, and `key_prefix`; and `next_step`, with `step` and `sentence`.
