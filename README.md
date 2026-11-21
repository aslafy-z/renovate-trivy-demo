# renovate-trivy demo

A small repository that shows [renovate-trivy](https://github.com/aslafy-z/renovate-trivy)
working with the hosted Renovate app.

Renovate is configured to open pull requests **only on 1 January**
(`"schedule": ["* * 1 1 *"]`), so on its own it would leave every update below
waiting in the Dependency Dashboard. renovate-trivy scans the images, finds
which updates remove critical or high CVEs, and tells Renovate to open those
now.

## What to look at

| Image (`k8s/`) | Update Renovate proposes | Expected |
|---|---|---|
| `lscr.io/linuxserver/prowlarr:2.7.0` | `3.0.2` | PR opened now, labelled `security`, CVE note in the body |
| `docker.io/valkey/valkey:9.2-alpine@sha256:cdcb…` | newer digest of the same tag | same: a digest-only update that fixes CVEs |
| `quay.io/argoproj/argocd:v3.6.1` + `haproxy:3.4.7-alpine` | one grouped `edge` branch | PR opened now for both: argocd fixes a CVE, haproxy rides along |
| `rajnandan1/kener:4.2.1` | `v4.2.2` | **stays** under "Awaiting Schedule": the update fixes nothing |
| `nginx:1-alpine` | none | only in the report |

- Pull requests: [labelled `security`](../../pulls?q=is%3Apr+label%3Asecurity)
- Renovate's [Dependency Dashboard](../../issues?q=is%3Aissue+Dependency+Dashboard), with a link to the report in its header
- The [CVE report](https://github.com/aslafy-z/renovate-trivy-data/blob/main/repos/aslafy-z/renovate-trivy-demo/report.md)
  and the [generated preset](https://github.com/aslafy-z/renovate-trivy-data/blob/main/renovate-trivy.json)
  in the data repository

## How it is wired

- [`.github/workflows/renovate-trivy.yml`](.github/workflows/renovate-trivy.yml)
  runs daily: a read-only `scan` job (Renovate lookup + Trivy), then a
  `publish` job that pushes to
  [aslafy-z/renovate-trivy-data](https://github.com/aslafy-z/renovate-trivy-data)
  with the `RENOVATE_TRIVY_DATA_TOKEN` secret (a fine-grained token with
  Contents read/write on the data repository only).
- [`renovate.json`](renovate.json) extends
  `github>aslafy-z/renovate-trivy-data:renovate-trivy`, the generated preset.

Nothing here calls the GitHub API on Renovate's behalf: the labels, the PR
notes and the early PRs are all Renovate's own doing, driven by the preset.
