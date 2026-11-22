# renovate-trivy demo

A small repository that shows [renovate-trivy](https://github.com/aslafy-z/renovate-trivy)
working with the hosted Renovate app.

Renovate is configured to open pull requests **only on 1 January**
(`"schedule": ["* * 1 1 *"]`), so on its own it would leave every update below
waiting in the Dependency Dashboard. renovate-trivy scans the images, finds
which updates remove critical or high CVEs, and tells Renovate to open those
now.

## What to look at

| Image | Update Renovate proposes | Expected |
|---|---|---|
| `lscr.io/linuxserver/prowlarr:2.7.0` | `3.0.2` | PR opened now, labelled `security`, CVE note in the body |
| `docker.io/valkey/valkey:9.2-alpine@sha256:cdcb…` | newer digest of the same tag | same: a digest-only update that fixes CVEs |
| `quay.io/argoproj/argocd:v3.6.1` + `haproxy:3.4.7-alpine` | one grouped `edge` branch | PR opened now for both: argocd fixes a CVE, haproxy rides along |
| `registry.k8s.io/kueue/kueue:v0.20.6` in `helm/kueue/values.yaml` | detected by the **helm-values** manager | scanned like any other image |
| `ghcr.io/cloudnative-pg/postgresql:17.11` in `k8s/database.yaml` | detected by a **custom regex** manager (`imageName:`) | scanned like any other image |
| `rajnandan1/kener:4.2.1` | `v4.2.2` | **stays** under "Awaiting Schedule": the update fixes nothing |
| `nginx:1-alpine` | none | only in the report |

- Pull requests: [labelled `security`](../../pulls?q=is%3Apr+label%3Asecurity)
- Renovate's [Dependency Dashboard](../../issues?q=is%3Aissue+Dependency+Dashboard), with the CVE summary at the top
- The [CVE report](https://github.com/aslafy-z/renovate-trivy-data/blob/data/repos/aslafy-z/renovate-trivy-demo/report.md)
  and the [generated preset](https://github.com/aslafy-z/renovate-trivy-data/blob/data/renovate-trivy.json)
  in the data repository

## How it is wired

This repository contains **no workflow and no token**. All it does is extend
the generated preset in [`renovate.json`](renovate.json):

```json
"extends": ["config:recommended", "github>aslafy-z/renovate-trivy-data:renovate-trivy#data"]
```

The scanning happens in [aslafy-z/renovate-trivy-data](https://github.com/aslafy-z/renovate-trivy-data):
its workflow clones this repository, asks Renovate (read-only) what it tracks
and proposes, scans the images with Trivy and publishes the preset, the report
and the SBOM archive to its own `data` branch.

Nothing calls the GitHub API on Renovate's behalf: the labels, the PR notes,
the early PRs and the summary at the top of the Dependency Dashboard are all
Renovate's own doing, driven by the preset.
