# Deploy flow

Call the tools in this order.

1. Project and app: call `deploy` with no app_id. That creates the app and queues its first deploy.
2. Database: call `database`.
3. Env: call `env`.
4. Deploy: call `deploy` with the app_id returned in step 1. That redeploys the same app. Do not create it again.
5. Poll status: call `status`.
6. Domain: call `domains`.

Without app_id it creates a project when needed, creates an app from a public Git URL, then queues a deploy.

`database` create looks up the project by name.
