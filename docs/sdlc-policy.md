# Secure SDLC policy

One page. Applies to every repository in this organization except upstream forks and archived repos.

## What runs, on every push and PR to main

A single org-level reusable workflow (`.github/workflows/security.yml` in this repo), pulled in by a ~15-line caller per repository. It auto-detects the stack; callers need no inputs.

| Surface | Tools |
|---|---|
| Dependencies + IaC + secrets (all repos) | Trivy (vuln/misconfig/secret), Gitleaks |
| SAST | CodeQL (Go, JS/TS) |
| Go | govulncheck, gosec |
| Node/TS | npm/pnpm audit |
| Helm charts | helm lint, kube-linter |
| Supply chain | Syft SBOM (SPDX artifact on every run); optional Trivy image scan + cosign keyless signing per release repo |

## Gating policy

**Report-only first, gate once baselined.** All scanners currently report (SARIF to the repo Security tab where available, logs otherwise) without failing builds. Once a repo's findings are triaged to a clean baseline, gating is enabled per repo by flipping the documented exit-code/`|| true` knobs in the reusable workflow call path. Newly created repos start report-only.

## Rollout

Pilot (validated end-to-end on a Go + Helm-chart repo) → org-wide wave (complete). New repos: copy the canonical caller, including its `permissions` block (`contents: read`, `security-events: write`, `actions: read`, `pull-requests: read`).

## Exclusions

Upstream forks (e.g. `opentelemetry-collector-contrib`, `kagent`, `plumbing`, `provider-runtime`) — scanned upstream; we avoid diverging their CI. Archived repos.

## Follow-ups

- Enable cosign keyless signing (the commented `sign` job) for image-publishing repos.
- Consider org-level OpenSSF Scorecard for a public posture score.

## Maintenance

The reusable workflow is versioned in this repo; changes to it roll out to all callers on their next run. Tool versions are pinned and bumped here, never per caller. SARIF upload steps tolerate repos without code-scanning entitlement (private repos), keeping runs report-only rather than red.
