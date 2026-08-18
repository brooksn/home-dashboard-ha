# Home Dashboard

Home Dashboard is a fullscreen household calendar display for a trusted local-network Android tablet.

## Install

1. Configure Home Assistant with credentials for the private `ghcr.io` registry.
2. Add `https://github.com/brooksn/home-dashboard-ha` in **Settings → Apps → App Store → ⋮ → Repositories**.
3. Install **Home Dashboard**, enter the required masked options, and start it.
4. Open `http://<home-assistant-host>:8088/` on the trusted LAN.

The app stores its SQLite database and session-secret material in its Supervisor-managed `/data` directory, so they persist through restarts and image updates.

## Security

- This app deliberately serves plain HTTP only on a trusted LAN. Do not expose port 8088 to the internet or through a public tunnel.
- Google Calendar credentials and passwords are stored as Home Assistant masked options and are never part of this repository or the browser bundle.
- The image is private; Home Assistant must be authenticated to `ghcr.io` before installation.

## Updates

Refresh the App Store after a new version is published. Home Assistant will pull the matching private image and retain `/data`.
