# Recipes for Node frameworks

Use the recipe for the framework the repository uses, before the first `deploy` call. Each recipe gives the files to read, the fix for each pre-flight reason that the framework can trigger, the `deploy` call, and how the app gets `DATABASE_URL`.

Every app in these recipes listens on port 3000 when PORT is not set. Astris does not set PORT. So omit `port` in the `deploy` call, or pass the port that the app's own code falls back to.

The default builder uses Node 18 when the repository does not ask for another major version. Set the engines field in package.json to the major version that the framework needs, as each recipe shows.

## Next.js

### Files to check

Read these files: package.json for the build and start scripts and the next version, the lockfile beside it, next.config (js, mjs or ts), a Dockerfile at the repository root if there is one, and .env.example.

### Pre-flight fixes

If Binds says package.json runs a dev server, set the start script to next start.

If Binds says package.json binds localhost, remove the -H option from the start script. By default, next start listens on every interface.

If Start says package.json pins a port, remove the -p option. next start reads PORT and falls back to 3000.

If Standalone says to set output to standalone, set it in next.config. This fix applies only when a root Dockerfile copies .next/standalone.

If Lockfile says there is no lockfile or more than one, commit the one lockfile of the package manager the repository uses.

```json file=package.json
{"scripts": {"build": "next build", "start": "next start"}, "dependencies": {"next": "^16", "react": "^19", "react-dom": "^19"}, "engines": {"node": "22"}}
```

### Deploy call

Without a Dockerfile, call `deploy` with no `build` and no `port`. The default builder runs the build script, then the start script.

With a root Dockerfile that copies .next/standalone, call `deploy` with `build` `dockerfile`, the `dockerfile` file name, and the `port` the Dockerfile sets. The standalone server listens on HOSTNAME. So in that Dockerfile, set HOSTNAME to the any-address that the Binds reason names, as the Next.js Dockerfile example does.

```js file=next.config.mjs case=standalone
export default { output: "standalone" };
```

```dockerfile file=Dockerfile case=standalone
FROM node:22-alpine AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build
FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production
ENV HOSTNAME=<any-address>
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

Replace <any-address> with the any-address that the Binds reason names. With this Dockerfile, pass `port` 3000.

### Database

Read DATABASE_URL only in server code. Never put it in a variable whose name starts with NEXT_PUBLIC_, because Next.js copies those variables into the browser bundle.

Then set DATABASE_URL as the last section says.

## Express

### Files to check

Read these files: package.json for the start script and the express version, the lockfile beside it, the entry file the start script runs, and .env.example.

### Pre-flight fixes

If Start says package.json has no production start, add a start script that runs the entry file with node.

If Binds says package.json runs a dev server, replace nodemon with node in the start script.

If Start says a file listens on a fixed port, read PORT and fall back to 3000, as below.

If Binds says a file binds 127 or localhost, remove the host argument from the listen call. Node then listens on every interface.

If Lockfile says there is no lockfile or more than one, commit the one lockfile of the package manager the repository uses.

```json file=package.json
{"scripts": {"start": "node server.js"}, "dependencies": {"express": "^5"}, "engines": {"node": "22"}}
```

```js file=server.js
const express = require("express");
const app = express();
app.get("/", (req, res) => res.send("ok"));
app.listen(process.env.PORT || 3000);
```

### Deploy call

Call `deploy` with no `build` and no `port`.

### Database

Pass DATABASE_URL to the database client. For the pg package, create new Pool with connectionString set to process.env.DATABASE_URL.

Then set DATABASE_URL as the last section says.

## NestJS

### Files to check

Read these files: package.json for the build, start and start:prod scripts and the @nestjs/core version, the lockfile beside it, the main.ts file that calls NestFactory.create, and .env.example.

### Pre-flight fixes

If Start says package.json has no production start, set the start script to node dist/main and the build script to nest build, and keep the dev dependencies that the project already has. The default builder runs start, not start:prod.

If Start says main.ts listens on a fixed port, read PORT and fall back to 3000, as below.

If Binds says main.ts binds 127 or localhost, remove the host argument from app.listen.

If Lockfile says there is no lockfile or more than one, commit the one lockfile of the package manager the repository uses.

```json file=package.json
{"scripts": {"build": "nest build", "start": "node dist/main"}, "dependencies": {"@nestjs/core": "^11", "@nestjs/common": "^11", "@nestjs/platform-express": "^11"}, "engines": {"node": "22"}}
```

```ts file=main.ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

### Deploy call

Call `deploy` with no `build` and no `port`.

### Database

Pass DATABASE_URL to the ORM that the repository uses. Prisma reads it from its schema file. TypeORM takes it as url.

Then set DATABASE_URL as the last section says.

## Lockfile and environment variables

These reasons can come from any recipe.

If Lockfile says the lockfile disagrees with packageManager, run the install of the package manager that packageManager names, then commit the new lockfile and delete the other one.

If Lockfile says a direct dependency is missing from the lockfile, run the package manager's install so the lockfile lists that dependency, then commit the lockfile.

If Lockfile says package-lock.json is not valid JSON, run npm install to write it again, then commit it.

If Lockfile says the lockfile is too large to check, check it yourself: it must list every dependency in package.json. This reason stays fixable, so tell the user that pre-flight could not check the lockfile.

If Environment variables says a file reads a name that .env.example does not list, add that name to .env.example with no value, and set its value with `env`.

## DATABASE_URL

Create the database with `database`, `action` `create`, `kind` `postgres`, and the `project` and `name`, as the deploy flow says. Build DATABASE_URL from the `user`, `password`, `host`, `port` and `database` that call returns:

```text
postgresql://<user>:<password>@<host>:<port>/<database>
```

The password has letters and digits only, so it needs no escaping.

Set DATABASE_URL with `env` only. `env` replaces every variable on the app, so send every other variable that the app reads in the same call. Then deploy again, as the deploy flow says.

Never write DATABASE_URL, a password or a key into a file in the repository. If the repository has .env.example, list the name DATABASE_URL there with no value.
