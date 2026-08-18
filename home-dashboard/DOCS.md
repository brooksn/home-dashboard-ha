# Home Dashboard

Home Dashboard is a fullscreen household calendar display for a trusted local-network Android tablet.

## Install

1. Configure Home Assistant with credentials for the private `ghcr.io` registry.
2. Add `https://github.com/brooksn/home-dashboard-ha` in **Settings → Apps → App Store → ⋮ → Repositories**.
3. Before saving the app configuration, generate a Google Calendar read-only refresh token with the OAuth bootstrap helper from the private source checkout. A Google OAuth client JSON supplies the client ID and secret, not the refresh token.
4. Install **Home Dashboard**, enter the required masked options (viewer password, administrator password, Google client ID, Google client secret, and Google refresh token), and start it. The OpenAI key and model are optional.
5. Open `http://<home-assistant-host>:8088/` on the trusted LAN. For Android browser PWA installation, configure the private Tailscale HTTPS endpoint below instead.

The app stores its SQLite database and session-secret material in its Supervisor-managed `/data` directory, so they persist through restarts and image updates.

## Security

- This app deliberately serves plain HTTP only on a trusted LAN. Do not expose port 8088 to the internet or through a public tunnel.
- Google Calendar credentials and passwords are stored as Home Assistant masked options and are never part of this repository or the browser bundle.
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
4. Restart the feature app and browse to `https://home-dashboard.<tailnet-name>.ts.net/` from a tablet connected to the tailnet. Do not enable Funnel.

The app configuration persists and is reapplied on Home Assistant restarts, so no SSH startup script is needed. The original `:8088` endpoint remains plain HTTP and LAN-only.

## Updates

Refresh the App Store after a new version is published, then select Update. Home Assistant will pull the matching private image and retain `/data`.
