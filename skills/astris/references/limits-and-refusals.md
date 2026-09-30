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

## Refusal sentences

| Status | Sentence |
| --- | --- |
| 401 | This key was refused. Put a current Astris API key in the MCP client config and try again. |
| 404 | Nothing with that id is on this account. Use the id from an earlier tool result. |
| 400 | Those arguments were refused. Check the tool description and send them again. |
| 400 | The field <field name> was refused. |
| 409 | That name or hostname is already taken. Choose another and try again. |
| 502 | The platform did not accept the change. Check status, then try again. |
| 500 | Astris failed on its side. Try again. |
| other | Astris failed on its side. Try again. |
| none | Those arguments were refused. Send project, name, git_url, and branch, or send app_id to redeploy. |
| none | app_id must be a non-empty string. |
| none | The Git URL must be a public http or https URL with no user or password. Private repositories are not available in this version. |
| none | Unknown tool. |
