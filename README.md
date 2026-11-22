# Umami on Render

Deploy [Umami](https://umami.is/), the open source, privacy-focused web
analytics tool, on Render with managed Postgres and Key Value (Redis).

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/render-examples/umami)

## What this creates

The Blueprint creates an `umami` project with a `production` environment:

| Resource | Type | Plan | Notes |
|---|---|---|---|
| `umami` | Web service (prebuilt image `ghcr.io/umami-software/umami:3`) | Starter | Health check at `/api/heartbeat` |
| `umami-db` | Render Postgres 17 | Basic-256mb, 5 GB | Internal connections only |
| `umami-cache` | Render Key Value | Starter | Internal connections only, `allkeys-lru` |
| `umami-settings` | Environment group | | Holds `APP_SECRET` |

The Blueprint sets `DATABASE_URL` and `REDIS_URL` from the datastores, and
generates a random `APP_SECRET` in the `umami-settings` group. Umami creates
its tables and runs migrations each time it starts, so there is no build step.
All data lives in Postgres, so the service needs no persistent disk.

## Deploy

1. Click **Deploy to Render** above, or create a Blueprint in the Render
   Dashboard that points at this repository.
2. When the deploy is live, open the service URL and log in right away with
   username `admin` and password `umami`. Umami makes you change the password
   before you continue.

> **Important:** Until you change the default password, anyone who opens the
> URL can log in as the admin.

3. Click **Settings > Websites > Add website**, enter your site's domain, and
   copy the tracking code into the `<head>` of your site:

   ```html
   <script defer src="https://<your-service>.onrender.com/script.js" data-website-id="<website-id>"></script>
   ```

## Optional settings

Add these to the `umami-settings` environment group in the Render Dashboard.
See the [Umami environment variables reference](https://docs.umami.is/docs/environment-variables)
for the full list.

| Variable | Purpose |
|---|---|
| `TWO_FACTOR_ENCRYPTION_KEY` | Required before any user can turn on two-factor authentication. Generate it with `openssl rand -hex 32`. |
| `TRACKER_SCRIPT_NAME` | Serves the tracker under another name than `script.js`, which some ad blockers block. |
| `COLLECT_API_ENDPOINT` | Moves the collection endpoint away from `/api/send`, for the same reason. |
| `DISABLE_TELEMETRY` | Set to `1` to turn off Umami's anonymous telemetry. |

Don't change `APP_SECRET` after you deploy. A new value signs everyone out
and counts returning visitors as new sessions for the current period.

## Custom domain

To serve analytics from your own domain, add a
[custom domain](https://render.com/docs/custom-domains) to the `umami`
service, for example `analytics.example.com`, and use it in the tracking code.
A first-party domain is also less likely to be blocked.

## Upgrades

The `:3` tag follows the latest Umami 3.x release. To pick up a new release,
trigger a manual deploy; Render pulls the image again and Umami runs any new
migrations on startup. To pin a version, change the tag (for example
`ghcr.io/umami-software/umami:3.5.0`). Read the
[release notes](https://github.com/umami-software/umami/releases) before you
move to a new major version.
