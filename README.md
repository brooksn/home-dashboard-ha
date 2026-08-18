# Home Dashboard Home Assistant Repository

This public repository contains only the Home Assistant App Store metadata for Home Dashboard. It contains no application source code, credentials, database contents, or container image layers.

The app image is privately distributed from `ghcr.io/brooksn/home-dashboard`; Home Assistant must have registry credentials before it can install or update the app.

Add this repository in **Settings → Apps → App Store → ⋮ → Repositories**:

```
https://github.com/brooksn/home-dashboard-ha
```

The private application source, release workflow, and image build context are maintained separately. This public repository is the only Git repository Home Assistant needs to fetch.

## Refresh Home Assistant after a release

The `Refresh Home Assistant app updates` workflow runs whenever the app metadata version changes.
It joins the tailnet briefly, asks Home Assistant to refresh `update.home_dashboard_update`, and
then disconnects. The app's existing automatic-update setting installs the new version once it is
discovered.

Before enabling the workflow, add these encrypted GitHub Actions secrets to this repository:

- `HOME_ASSISTANT_URL`: the private Home Assistant API origin, for example
  `http://homeassistant.<tailnet>.ts.net:8123`.
- `HOME_ASSISTANT_TOKEN`: a Home Assistant long-lived access token from an administrator account.
- `TS_OAUTH_CLIENT_ID` and `TS_AUDIENCE`: the client ID and audience for a Tailscale federated
  identity with writable `auth_keys` scope, restricted to `tag:ci`. No Tailscale
  client secret is stored in GitHub.

In the tailnet access policy, allow `tag:ci` to connect only to the Home Assistant
node on TCP port `8123`. The workflow stays a successful no-op until all four secrets are set.
