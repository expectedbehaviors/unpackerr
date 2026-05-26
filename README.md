# Unpackerr Helm Chart

Baseline Helm chart for [Unpackerr](https://unpackerr.zip) (extraction daemon for Radarr, Sonarr, Lidarr, Readarr; FLAC+CUE splitting for Lidarr). Built on the [bjw-s app-template](https://github.com/bjw-s/helm-charts).

## What this baseline does

- Uses a sanitized baseline: no real hostnames or environment-specific PVC names.
- Optional secret injection via **onepassworditem** (1Password → Kubernetes secret `unpackerr`); container uses `envFrom` so secret keys become env vars.
- Override persistence with `existingClaim` in your values to point at your download PVCs (same layout as *arr apps).

## Secrets

No secrets are required for baseline startup. The app will run but cannot talk to *arr APIs until the `unpackerr` secret exists with keys such as:

- `UN_LIDARR_0_API_KEY`
- `UN_RADARR_0_API_KEY` (optional)
- `UN_SONARR_0_API_KEY` (optional)
- `UN_READARR_0_API_KEY` (optional)

When `onepassworditem.enabled=true`, set `onepassworditem.items` to your 1Password item(s) whose **field names** match the env var names above. The operator syncs them into secret `unpackerr` in the release namespace.

## Key values

| Area | Where | Default |
|------|-------|---------|
| Secret source | `onepassworditem.enabled` | `false` |
| 1Password items | `onepassworditem.items` | `[]` |
| Ingress | `unpackerr.ingress.main.enabled` | `false` |
| Config storage | `unpackerr.persistence.config.*` | PVC 1Gi |
| Downloads | `unpackerr.persistence.downloads.*` | `emptyDir` (override with `existingClaim` in instantiation) |
| *arr URLs | `unpackerr.controllers.main.containers.main.env.UN_*_0_URL` | In-cluster DNS (e.g. `lidarr.lidarr.svc.cluster.local`) |

## Install

From Helm repo (after chart is published):

```bash
helm repo add expectedbehaviors https://expectedbehaviors.github.io/unpackerr
helm install unpackerr expectedbehaviors/unpackerr -f my-values.yaml -n unpackerr --create-namespace
```

## Render & validation

```bash
helm dependency update . && helm template unpackerr . -f values.yaml -n unpackerr
helm template unpackerr . -f values.yaml --set onepassworditem.enabled=true -n unpackerr
```

## Publishing (maintainers)

This chart is published to **https://expectedbehaviors.github.io/unpackerr** via GitHub Actions:

1. **Release on merge to main** — On push to `main` (or manual dispatch), lints the chart and creates a release with tag `unpackerr-v<version>`.
2. **Helm chart publish** — Runs on that release (or manual dispatch) and publishes the package to the repo’s GitHub Pages. Pull from the repo with `helm repo add expectedbehaviors https://expectedbehaviors.github.io/unpackerr`.
3. **Enable GitHub Pages** for the repo: **Settings → Pages → Source: Deploy from a branch** (or **GitHub Actions** if the publish action deploys to Pages). Branch: **gh-pages** (or as configured by the action).

## Argo CD

Point your Application at this repo (path: `.`) and pass your values. Namespace typically `unpackerr`. Override persistence with `existingClaim` for config and download PVCs to match your environment.
