# Troubleshooting a deploy

Use this file when `diagnose` returns a cause, when a pre-flight row is not `pass`, or when a deploy stops answering.
Do the action it gives in the same session. Only an entry that tells the user and stops ends the session.

## A diagnosis with a cause

The `diagnosis` object holds `cause`, `reason`, `evidence` and `next_step`, and `hint` when a log line matched.

### `deployment_failed`

The newest deploy failed before the app started.
Read `evidence.log_tail` and follow the branch that `next_step` and `reason` name.
When the repository failed to clone over http, check that the Git URL and branch are right and that the repository is public, then call `deploy` with the `app_id`.
When the repository failed to clone over SSH, check that the repository has the deploy key with read access, then call `deploy` with the `app_id`.
When `log_tail` has a line that starts `Astris stopped this step after`, the step ran past the Astris time limit and was stopped. When `reason` names a build, make the build shorter in the repository (for example, remove a slow step), commit and push it, then call `deploy` with the `app_id`. When `reason` names cloning, tell the user the repository takes too long to download, and stop.
When the Dockerfile build or the Nixpacks build failed, fix the first error in `log_tail` in the repository, commit and push it, then call `deploy` with the `app_id`.
When `reason` names no step, find the first error in `log_tail`, fix it the same way, then call `deploy` with the `app_id`.

### `out_of_memory`

The app was stopped for using more memory than its limit.
When `next_step` says the pool memory is full, give the user that sentence and stop: only the user can free the account's memory.
Otherwise, read `log_tail`, reduce the app's memory use in the repository, commit and push it, then call `deploy` with the `app_id`.

### `app_exited`

The app started and stopped.
When `next_step` says the pool memory is full, give the user that sentence and stop: only the user can free the account's memory.
When the exit code is not zero, read `log_tail`, fix the cause in the repository, commit and push it, then call `deploy` with the `app_id`.
When the exit code is zero, `next_step` says no change is required: tell the user the app stopped cleanly and stop, because a redeploy alone changes nothing.

### No recognised cause

`recognised` is `false` when the app reports an error and the evidence does not name a cause.
Read `evidence.log_tail` (the newest deployment's log) and `evidence.deployments`, then call `logs` with the `app_id` and no `source` to read what the app printed while running, and fix what those lines show; commit and push the fix, then call `deploy` with the `app_id`.
When the lines show nothing you can act on, tell the user what they show and stop.

### No diagnosis

`diagnosis` is `null` and `note` says what was checked.
When `note` says the app has been running, open the app's address and read `logs` when it does not answer; do not redeploy.
When `note` says the app has not been running for 60 seconds yet, call `diagnose` again after that.
When `note` says the newest deploy did not fail and the app was not checked, call `status` with the `app_id` and `preflight` with the repository's files before any redeploy.

## A hint

Each bullet starts with the id of the hint row that matched a log line; the result carries that row's sentence.
Use the bullet to decide the change. A change to the repository ships only after you commit and push it, so commit and push it before each `deploy` the bullet names.

- `out_of_memory`: reduce the app's memory use in the repository; when it cannot use less, tell the user it needs a plan with more memory.
- `dockerfile_missing`: add a Dockerfile at the path the app uses, then call `deploy` with the `app_id`.
- `nixpacks_no_plan`: add a Dockerfile, or a package.json or requirements.txt at the repository root, then call `deploy` with the `app_id`.
- `nixpacks_no_start`: add a start script to package.json, or a Procfile, then call `deploy` with the `app_id`.
- `lockfile_out_of_sync`: run the install, commit the lockfile, then call `deploy` with the `app_id`.
- `missing_env`: read the current names with `env_names`, then set the variable with `env` together with every other variable the app needs, because `env` replaces the whole set, then call `deploy` with the `app_id`.
- `node_module_missing`: put the package in dependencies, not devDependencies, or fix the import path, then call `deploy` with the `app_id`.
- `python_module_missing`: add the package to requirements.txt, then call `deploy` with the `app_id`.
- `command_not_found`: add the program to dependencies, or fix the command, then call `deploy` with the `app_id`.
- `permission_denied`: make the script executable in the repository, or write temporary files to /tmp, then call `deploy` with the `app_id`.
- `port_in_use`: start one server only, on the app's port, then call `deploy` with the `app_id`.

## A pre-flight row that is not `pass`

Read each row's `reason` and do the action for its check.
When the verdict is `refuse`, fix the row and run `preflight` again; when the row cannot be fixed from the repository, tell the user and stop.

- Secrets `refuse`: a key shape is in a file you sent. Revoke the key in the service that issued it, remove it from the files and the git history, then run `preflight` again. After the app is created, read the current names with `env_names`, then set the new key with `env` together with every other variable the app needs, because `env` replaces the whole set.
- Secrets `did not run`: no key shape was found, and the check cannot read your git history. Search the repository's history for a committed key before the deploy.
- Secrets `fixable`: a .env file holds values that git can commit. Add the file to .gitignore. After the app is created, read the current names with `env_names`, then set those values with `env` together with every other variable the app needs, because `env` replaces the whole set.
- Never put a key value in a file you send, a log or a message; set it with `env`.
- Runtime `refuse`: the app depends on a builder-locked backend with no export path. Tell the user it cannot run here as it is, and stop.
- Runtime `fixable`: an edge runtime or a wrangler configuration runs only after a rewrite the user authorises. Ask the user before you rewrite it; when no runtime was detected, add a manifest or a Dockerfile.
- Binds `fixable`: change a bound host to every address, and replace a dev server start with a production start.
- Start `fixable`: add a production start that reads PORT, remove a fixed port, and use three or fewer workers.
- Environment variables `fixable`: list each name in .env.example and set every name with `env` in one call, because `env` replaces the whole set.
- Lockfile `fixable`: commit one lockfile that matches package.json and the package manager it names, then run `preflight` again.
- Standalone `fixable`: set output to standalone in next.config so the image can copy .next/standalone, then run `preflight` again.
