# Go live

Take an app from pre-flight to an address that answers, then fix or roll back when it does not. Every step is a call to the Astris MCP.

## Before you start

Call `whoami` and read `next_step.step`, as SKILL.md says. Call `usage` to see the room left.

## Pre-flight

Call `preflight` with the repository's files as `files`, as the deploy flow says. Stop on a `refuse` verdict. Fix each `fixable` reason, then commit and push the fix, because the app is built from its Git repository.

## Deploy and wait

Call `deploy` with `wait` set to `true`. For a new app, follow step 1 of the deploy flow and keep the `application_id` it returns as `app_id`.

With an SSH `git_url`, leave out `wait` on the create, because it returns a deploy key and no deploy runs. Add the key as the deploy flow says, then call `deploy` with the `app_id` and `wait` set to `true`.

When the `outcome` is `done`, go to Check the address. When it is `still running, check status`, poll `status` with the `app_id`, as step 5 of the deploy flow says. When it is `error` or `cancelled`, go to When the deploy fails.

## When the deploy fails

Call `diagnose` with the `app_id` and tell the user what the result says. When it has a `hint`, follow that sentence.

When `diagnose` cannot tell the cause, call `logs` with the `app_id` and `source` `deploy` for a build failure, or with no `source` for an app that stopped. Read the last lines for the error.

## The three common fixes

Pre-flight names each of these in its Binds or Start reason.

Port: the app must listen on the port it was created with. `port` can be set only at creation, so change the app's code, not the port. Make the app read PORT and fall back to that port. Astris does not set PORT, so when the app needs it, set PORT with `env` to that same value, and send every other variable the app reads in the same call.

Bind: the app must listen on every interface, not on localhost. Set its host to the any-address that the Binds reason names.

Start command: the app needs a production start, not a dev server. Add a start script to package.json, or a Procfile web line, that starts the server in production.

Make the fix in the repository, commit and push it, then redeploy.

## Redeploy

Call `deploy` with the `app_id` and `wait` set to `true`. This redeploys the same app. Do not create a second one. Then read the `outcome` again, as in Deploy and wait.

## Check the address

Call `status` with the `app_id`. The newest item in `deployments` must have `status` `done`, and the application's `status` must not be `error`.

Call `logs` with the `app_id` and no `source`. A runtime error or a crash in the last lines means the app is not serving; go to When the deploy fails.

The first time, call `domains` with the `app_id`. This binds the address. Keep its `hostname` and `domain_id`.

On a later check, call `domains` with the `app_id`, `action` `status` and the `domain_id` instead, because a second bind is not needed and this changes nothing.

Give the user the address https://<hostname> and ask them to open it and tell you what they see. Astris does not check whether the app answers requests.

A worker or a cron app has no address by design. Its check ends with `status` and `logs`.

## Roll back

When a redeploy broke an app that worked, and the user asks you to go back, call `rollback` with the `app_id`. It goes back to the last working deploy. To go back to a chosen deploy, also pass `commit`, 7 to 40 hex characters of a commit from the last ten deploys.

Variables return to that deploy's values until the next deploy.

If `rollback` answers "Rollback is not available on this server.", tell the user, and fix forward with the steps above instead.
