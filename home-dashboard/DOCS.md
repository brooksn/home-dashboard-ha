# Home Dashboard

Home Dashboard is a fullscreen, private, read-only Google Calendar display for a household tablet.

## Install

1. Configure Home Assistant with credentials for the private `ghcr.io` registry.
2. Add `https://github.com/brooksn/home-dashboard-ha` in **Settings → Apps → App Store → ⋮ → Repositories**.
3. In Google Cloud, create a Web OAuth client, enable Google Calendar API, and register `https://home-dashboard.tailab0c1.ts.net/api/admin/google/oauth/callback` as its exact redirect URI.
4. Install **Home Dashboard**, enter the Google client ID/secret, and set `external_url` to `https://home-dashboard.tailab0c1.ts.net`. Add an OpenAI API key only for AI curation and daily summaries.
5. Open `https://home-dashboard.tailab0c1.ts.net/admin/calendar`, connect Google, select calendars, assign **Brooks personal**, **Brooks work**, or **Rose**, then save.

The app stores its database, refresh tokens, selected calendars, and cached events in Supervisor-managed `/data`. Nearby, back-to-back, and overlapping events for the same person are shown as one busy block, even when AI processing is unavailable.

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

Each dashboard release publishes a signed private image. When the source repository's metadata-handoff credentials are configured, it also updates this app metadata and requests a private Home Assistant update scan. With automatic updates enabled, Home Assistant pulls the matching image and retains `/data`; otherwise use **Settings → System → Updates → Check for updates**.
