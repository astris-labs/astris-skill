# Recipes: static sites, workers and cron jobs

Use the recipe that matches the repository, then follow the [Deploy flow](deploy-flow.md).

## Static site

A site of plain HTML, CSS and JS files with no build step takes `build` `static`. It is served on port 80, and `port` is ignored.
The static build copies the files of the repository as they are, and runs no build command. If `index.html` is not at the repository root, pass the folder that holds it as `publish_directory`.
Pass `spa` `true` when the site routes in the browser, so that a path that is not a file serves `index.html`.
A Vite app built with React, Vue, Svelte, Preact, Lit, Solid or Qwik, and with no `start` script, takes no `build` and no `port`. The default builder runs `vite build` and serves the output on port 3000. Do not use the static build for it, because the static build does not run `vite build`.
SvelteKit, Remix and React Router framework apps are not served by the default builder this way: give them a `start` script, or deploy them with `build` `dockerfile`, as pre-flight says.
Any other site generator that runs from a `build` script in package.json, such as Astro, deploys with this Dockerfile at the repository root, with `build` `dockerfile`, `dockerfile` `Dockerfile` and `port` `80`. Change `dist` to the folder that the generator writes.

```dockerfile
FROM node:lts AS build
WORKDIR /app
COPY . .
RUN npm i
RUN npm run build

FROM httpd:2.4 AS runtime
COPY --from=build /app/dist /usr/local/apache2/htdocs
EXPOSE 80
```

## Background worker

A process that runs until it is stopped and serves no web page takes `kind` `worker` and the `command` that starts it, such as `node worker.js`. A job that runs and ends belongs in a cron app.
The default builder still needs a start command it can find, so set the `start` script in package.json to the same command.
A worker has no address. Step 6 of the deploy flow does not apply.

## Cron job

A command on a timer takes `kind` `cron`, a `command` and a `schedule`.
The schedule runs the command with bash inside the running app, with the files and variables of the app.
Each run is stopped after 600 seconds, and only the first 1 MiB of its output is kept. A run that is stopped or cut off counts as failed.
The schedule needs `bash`, `sleep`, `head` and `wc` in the app's image. Debian and Ubuntu images have them; an Alpine image needs `apk add bash`. Without one of them, each run fails at once.
A cron app must keep running, so its start must not end. Set `"start": "sleep infinity"` in package.json, put the job in its own file, such as `job.js`, and pass `node job.js` as the `command`.
The `schedule` is five fields separated by single spaces: minute, hour, day of month, month, day of week. Each field uses digits, `*`, `,`, `-` and `/`. Names such as `MON`, shortcuts such as `@daily`, and a sixth field are refused.
The schedule runs in UTC. `0 3 * * *` runs at 03:00 UTC on each day, and `*/15 * * * *` runs at minute 0, 15, 30 and 45 of each hour.
A cron app has no address. Step 6 of the deploy flow does not apply.
