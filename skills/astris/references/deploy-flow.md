# Deploy flow

Read the repository first, then call the tools in this order.

Before the first deploy, find out from the repository the port the app listens on, whether it has a Dockerfile in its top folder, whether it needs a database, whether it needs a bucket, and which environment variables it reads.

Astris builds every app from the top folder of its repository; `build_path` other than `/` and a `dockerfile` with a folder in it are refused. When the app sits in a sub-folder, put a Dockerfile in the top folder that copies that sub-folder and builds it, or move the app to its own repository.

Before you create the app, call `preflight` with the repository's files as `files`, each with a `path` and its whole contents as `text`.

When the app already exists, read its variable names with `env_names` and send them to `preflight` as `env_names`, so the check can name what .env.example lists and the deploy does not set. Send names only, never a value. A first deploy has no names to send.

Send whole files only, and never cut a file short.

If the repository does not fit the pre-flight limits in one call, send the build and start files first, such as package.json, requirements.txt or pyproject.toml, the lockfile at the repository root, the Dockerfile and any .env.example, then the app's own source code, starting with its entry point, until the next file would go over a limit.

Leave out dependencies, build output, tests and docs.

If the verdict is `refuse`, stop and tell the user the reasons.

If the verdict is `fixable`, fix each fixable reason or tell the user about it, then continue with the create.

Commit and push any fix before the create, because the app is built from its Git repository.

For each `did not run` row, read its reason and do that check yourself before creating the app.

Secrets reads the files you send for known key shapes. If it finds one, it answers refuse. If it finds none, it reports did not run, because the API does not read your git history. Lockfile reports did not run when you send no lockfile.

The Secrets check looks for a service_role key, a secret key that starts with sb_secret_, a live secret key that starts with sk_live_, a restricted key that starts with rk_live_, an access key ID that starts with AKIA or ASIA, and a private key.

The verdict covers only the files you sent and the checks that ran.

If some files did not fit in the call, tell the user that pre-flight read only part of the repository.

1. Create the app. First use `list` with `projects` to read the project names on this account, and pass the exact name of the project the app belongs in; a name that matches none creates a new project. Then call `deploy` with no `app_id` and the app's Git URL. Choose the build: omit `build` when the repository has no Dockerfile and is not a plain HTML, CSS and JS site, because the default builder needs no Dockerfile; pass `build` `dockerfile` with the `dockerfile` file name when the repository has one in its top folder; pass `build` `static` for an HTML, CSS and JS-only site, which is served on port 80 and ignores `port`. Pass `port` when the repository shows the port the app listens on; without `port`, an app that is not static is created on port 3000. An app that serves no web page takes `kind`, `command` and `schedule`; the part below says how. Port, build, Dockerfile and kind can be set only at creation, so if the repository does not show the port, ask the user before this step. This creates the project if needed, creates the app and queues its first deploy. Keep the `application_id` it returns, and pass it as `app_id` to later tools.
2. Database, only when the app needs one: call `database` with the project name. It returns the host, port, user and password, and the database name when `database` is a string. When `kind` is `redis`, `database` is null and there is no database name. Do not set a variable from that null. The app does not receive these until step 3.

Astris offers Postgres and Redis databases. Set kind to postgres or redis.

3. Variables, when the app reads any, and always after step 2 created a database: call `env` with the `app_id`. Set each value under the variable name the app's own code reads for it, including the five database values from step 2. The database port is not the port the app listens on; never write it into the app's own port setting.
4. Redeploy, only when a database was created or variables were set: call `deploy` with the `app_id` from step 1. This redeploys the same app. Do not create a second one.
5. Poll status: call `status` with the `app_id`. `deployments` is newest first, at most five items, and each item has `status`, `created_at` and `finished_at`. Repeat this step until the first item counts: it counts once you have seen that item, identified by its `created_at`, with `status` `running` during this step. An empty list, a null `status`, an application `status` of `idle`, or a first item you have not seen `running` means the new deploy has not shown yet, so repeat this step. The deploy has finished when the first item counts, its `status` is `done`, and the application's `status` is not `error`; then go on to step 6. The deploy has failed when the first item counts and its `status` is `error`; tell the user the application's `status` and that item's `status`, `created_at` and `finished_at`, then stop. If the first item counts and its `status` is `cancelled`, or the application's `status` is `error` while the first item counts and is not `running`, tell the user the values you read and stop; neither is finished or failed. After 24 repeats without a finished or failed result, tell the user the deploy has not finished yet, give the values you read, and stop.
6. Address, for an app that serves a web page only: the app got its astris.live address when step 1 created it. Read `hostnames` from `status` for the `app_id` and give the address to the user. Only to add the user's own domain, call `domains` with the `app_id` and the `hostname`. A worker or a cron app has no address by design: its flow ends after step 5, and you tell the user that.

## Waiting for a deploy

To wait for a deploy in the same call, pass `wait` set to `true` with the `deploy` of step 4. When step 4 does not apply, pass it with the `deploy` of step 1 instead. The result's `outcome` is `done`, `error`, `cancelled` or `still running, check status`.

When it is `done`, go on to step 6. When it is `error`, `diagnose` with the `app_id` says why; follow [Troubleshooting](references/troubleshooting.md) for that cause and act on it yourself, and when it names a cause you cannot fix, stop and tell the user. When it is `cancelled`, stop and tell the user. When it is `still running, check status`, go on with step 5.

`wait` does not apply to an SSH `git_url`.

## Workers and scheduled jobs

An app that serves no web page deploys through the same steps, with the fields below set at creation. They cannot be set later.

Pass `kind` `worker` with a `command` for a process that runs all the time and serves no web page. The `command` runs as the app's process, and the app has no public address.

Pass `kind` `cron` with a `command` and a `schedule` for a command on a timer. The `schedule` is five space-separated fields, for example `0 3 * * *`. A cron app must keep running: it keeps its normal start, and the schedule runs the command inside it.

Omit `kind` for an app that serves a web page, and send no `command` and no `schedule` with it. A worker or a cron app cannot use a static build.

A worker and a cron app get no hostname. Step 6 does not apply to them: the flow ends after step 5, and you tell the user that the app runs with no address by design.

## Private repository

A private repository deploys through SSH. Pass the SSH URL as `git_url`; the create returns a `deploy_key` with `type`, `public_key` and `fingerprint`, and no deploy runs. Add that public key to the repository as a read-only deploy key, then deploy again with the returned `application_id` as `app_id` to queue the first deploy.

A Git host must be a public name, and every address it resolves to must be public. An IP address, localhost, or an internal name is refused, and so is a name that does not resolve.

## Builder's own domain

A custom domain binds through `domains` with `hostname` instead of `slug`. The bind returns `domain_id`, `status` and `verification` records. Give the user the CNAME and TXT values to add to their DNS, then pass `action` `verify` and the `domain_id` to check that DNS is ready.

## Push to deploy

A push-to-deploy URL comes from `deploy_hook` with the `app_id`. The URL is a secret; never show it in a log, a commit message or a public place. To wire up GitHub, open the repository, then Settings, Webhooks, Add webhook, put the URL in Payload URL, set Content type to application/json, and subscribe to push events.

## Storage buckets

An app that stores its own files needs a bucket. Find out from the repository which environment variables it reads for object storage before you create one.

Create the bucket with `storage`, action `create`, and the name the app or the user calls it. The result gives the `bucket_id` and the S3 credentials `endpoint`, `bucket`, `access_key_id`, `secret_access_key` and `region`. The secret key appears only in that result, so store it now.

Give the app the credentials as variables: with `env`, set the five values under the variable names the app's own code reads for them, together with the database values from step 2. Read the names back with `env_names`, which returns names and no values.

Delete a bucket only when it is empty: `storage`, action `delete`, with the `bucket_id` from an earlier result. A delete of a bucket that still holds objects is refused and removes nothing.

## Variable names

The names of an application's environment variables come from `env_names` with the `app_id`. No value is returned. Use `env` to set or replace the values.
