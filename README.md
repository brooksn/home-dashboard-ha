# Home Dashboard Home Assistant Repository

This public repository contains only the Home Assistant App Store metadata for Home Dashboard. It contains no application source code, credentials, database contents, or container image layers.

The app image is privately distributed from `ghcr.io/brooksn/home-dashboard`; Home Assistant must have registry credentials before it can install or update the app.

Add this repository in **Settings → Apps → App Store → ⋮ → Repositories**:

```
https://github.com/brooksn/home-dashboard-ha
```

The private application source, release workflow, and image build context are maintained separately. A release updates this repository's app version automatically when the source repository's scoped GitHub App credentials are configured; otherwise update `home-dashboard/config.yaml` manually. This public repository is the only Git repository Home Assistant needs to fetch.

## Refresh Home Assistant after a release

Every signed dashboard release creates a **Home Assistant** deployment in the private source repository. When its secrets are configured, the `Refresh Home Assistant app updates` workflow joins the tailnet briefly, refreshes `update.home_dashboard_update` until the expected version is visible, calls `update.install`, and waits until Home Assistant reports that version as installed. It then marks the source deployment successful or failed, with a link to this workflow's log.

Before enabling the workflow, add these encrypted GitHub Actions secrets to this repository:

- `HOME_ASSISTANT_URL`: the private Home Assistant API origin, for example
  `http://homeassistant.<tailnet>.ts.net:8123`.
- `HOME_ASSISTANT_TOKEN`: a Home Assistant long-lived access token from an administrator account.
- `TS_OAUTH_CLIENT_ID` and `TS_AUDIENCE`: the client ID and audience for a Tailscale federated
  identity with writable `auth_keys` scope, restricted to `tag:ci`. No Tailscale
  client secret is stored in GitHub.

In the tailnet access policy, allow `tag:ci` to connect only to the Home Assistant node on TCP port `8123`. The workflow is skipped until all four secrets are set.

## Deployment status reporting

Create a second GitHub App installed on **only** `brooksn/home-dashboard`. Grant it only repository **Deployments: Read and write** permission. In this metadata repository's Actions settings, add:

- Repository variable `DEPLOYMENT_STATUS_APP_CLIENT_ID`: the App's client ID.
- Repository secret `DEPLOYMENT_STATUS_APP_PRIVATE_KEY`: a generated private key for that App.

This App can update deployment status only; it cannot read or write source code. The workflow exchanges its private key for a short-lived installation token, then records `success` only after `update.home_dashboard_update` reports the requested version installed.
