# Home Dashboard

Home Dashboard is a fullscreen household calendar display for a trusted local-network Android tablet.

## Install

1. Configure Home Assistant with credentials for the private `ghcr.io` registry.
2. Add `https://github.com/brooksn/home-dashboard-ha` in **Settings → Apps → App Store → ⋮ → Repositories**.
3. In Google Cloud, create a Web OAuth client, enable Google Calendar API, and register `https://home-dashboard.tailab0c1.ts.net/api/admin/google/oauth/callback` as its exact redirect URI.
4. Install **Home Dashboard**, enter Google client ID/secret and `external_url` set to `https://home-dashboard.tailab0c1.ts.net`. The OpenAI key is optional; retained viewer/admin password options are ignored.
5. Open `/admin/calendar` at that HTTPS address, connect each household Google account, select calendars, then open the trusted LAN display URL on the tablet.

The app stores its SQLite database in its Supervisor-managed `/data` directory, so it persists through restarts and image updates.

## Backup and restore

Use Home Assistant's normal app backup flow to preserve `/data`, including selected calendars, cached materialized events, and refresh tokens. Restore the app backup before starting a replacement instance; if the database is intentionally discarded, reconnect Google accounts and select calendars again.

## Security

- This app has no application password: every route relies on the private Tailscale network. Do not expose port 8088 to the internet or through a public tunnel.
- Google client credentials remain Home Assistant masked options. OAuth refresh tokens live only in the mode-0600 `/data` database and never reach the browser bundle.
- The image is private; Home Assistant must be authenticated to `ghcr.io` before installation.

## Private Tailscale HTTPS endpoint

For a private, trusted HTTPS origin, use the [Tailscale with features](https://github.com/lmagyar/homeassistant-addon-tailscale) Home Assistant app. The stock Tailscale app cannot persistently proxy this dashboard's separate port 8088.

1. In the Tailscale admin console, enable **MagicDNS** and **HTTPS certificates** under **DNS**. In **Access controls**, permit the app's tag, for example:

   ```json
   "tagOwners": {
     "tag:home-dashboard": ["autogroup:admin"]
   }
   ```

2. Configure the feature app with the matching tag and a private service:

   ```yaml
   advertise_tags:
     - tag:home-dashboard
   services:
     - name: svc:home-dashboard
       target: http://127.0.0.1:8088
       protocol: https
       port: 443
       path: /
   ```

3. In the console's **Services** area, set the `svc:home-dashboard` required port to `tcp:443`, then approve the tagged Home Assistant node as service host if prompted. This is the TLS listener; the service target stays on `8088`.
4. Restart the feature app and browse to `https://home-dashboard.tailab0c1.ts.net/` from a tablet connected to the tailnet. Do not enable Funnel.

The app configuration persists and is reapplied on Home Assistant restarts, so no SSH startup script is needed. The original `:8088` endpoint remains plain HTTP and LAN-only.

## Updates

Refresh the App Store after a new version is published, then select Update. Home Assistant will pull the matching private image and retain `/data`.
