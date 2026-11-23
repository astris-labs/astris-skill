# Limits and refusals

## Limits

| Limit | Rule |
| --- | --- |
| Env keys | At most 200 keys. |
| Env total size | 64 KiB. A larger body is refused while it is read. |
| App name length | 1 to 63 characters. |
| Slug length | 1 to 63 characters, `[a-z0-9-]`, no leading or trailing `-`. |
| git_url length | Longer than 512 characters is refused. |
| Port | An integer from 1 to 65535. Omitted means 3000. |
| Command length | 1 to 512 characters, with no newline. |
| Schedule | Five space-separated fields, at most 100 characters. |
| Static site port | Served on port 80. The `port` you send is ignored. |
| Pre-flight files | At most 64 files, with each path sent once. |
| Pre-flight file size | At most 32 KiB of UTF-8 text per file, or 320 KiB for a package-lock.json, pnpm-lock.yaml, yarn.lock or bun.lock at the repository root. |
| Pre-flight request size | At most 512 KiB for the JSON request body. |
| Diagnose text | At most 50 lines, each at most 200 characters. |

A pre-flight call over any of these limits is refused whole and runs no check, so send fewer files instead of the same call again.

## Refusal sentences

| Status | Sentence |
| --- | --- |
| 401 | This key was refused. Put a current Astris API key in the MCP client config and try again. |
| 404 | Nothing with that id is on this account. Use the id from an earlier tool result. |
| 400 | Those arguments were refused. Check the tool description and send them again. |
| 400 | The field <field name> was refused. |
| 409 | That name or hostname is already taken. Choose another and try again. |
| 409 | This database has no backup job. Only Postgres databases are backed up. |
| 429 | This account asked for too many changes in a short time. Wait a little, then try again. |
| 502 | The platform did not accept the change. Check status, then try again. |
| 500 | Astris failed on its side. Try again. |
| other | Astris failed on its side. Try again. |
| none | Those arguments were refused. Send project, name, git_url, and branch, or send app_id to redeploy. |
| none | app_id must be a non-empty string. |
| none | The Git URL must be an https URL with no user or password, or an SSH URL for a private repository, as git@host:owner/repo.git or ssh://git@host/owner/repo.git. |
| none | confirm_name is required. Send the application's exact name. |
| none | The name does not match the application. |
| none | Unknown tool. |

When the reply gives a number of seconds, the rate-limit sentence is "This account asked for too many changes in a short time. Wait N seconds, then try again.", with N replaced by that number.
For a one-second wait, the sentence is "This account asked for too many changes in a short time. Wait 1 second, then try again."
If a sentence asks you to wait, wait the number of seconds it gives, or 60 seconds if it gives no number.
Then try the call once more.
If it still asks you to wait, tell the user and stop; do not retry in a loop.

## Storage refusals

| Status | Sentence |
| --- | --- |
| 409 | That bucket name is already taken. Choose another and try again. |
| 409 | This bucket is not empty, so nothing was deleted. Remove its objects first, then try again. |
| 401 | This key was refused. Put a current Astris API key in the MCP client config and try again. |
| 404 | Nothing with that id is on this account. Use the id from an earlier tool result. |
| 400 | Those arguments were refused. Check the tool description and send them again. |
| 503 | Storage is not available right now. Try again later. |
| 502 | The platform did not accept the change. Check status, then try again. |
| 500 | Astris failed on its side. Try again. |
| other | Astris failed on its side. Try again. |
| 403 | This account has reached its bucket limit. Ask the builder which bucket they no longer need, delete it, then create again. |

A refusal that names the bucket limit does not pass on a retry; tell the user what it says and stop.

## No plan

This account has no plan yet. Choose a plan at https://api.astris.run/console/billing, then try again.

## Over the pool

A refusal that says something does not fit this pool has one of two endings. Tell the user the sentence as it is, and never name a plan the sentence does not give.

RAM and CPU do not fit this pool. Change your plan at https://api.astris.run/console/billing. To get more room, move to <Plan>: ask your coding agent to switch you to <Plan>, or run astris billing switch --plan <plan>.

RAM and CPU do not fit this pool. Change your plan at https://api.astris.run/console/billing. You are on the largest plan we sell now. To ask for more room, write to support@astris.run.
