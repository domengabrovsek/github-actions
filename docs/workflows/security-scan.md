# `security-scan.yml`

Runs three scanners:

| Scanner | Checks | Job |
| --- | --- | --- |
| Gitleaks | Secrets, over full history | `Scan (gitleaks, bearer)` |
| Bearer | SAST. Fails on `fail_severity`, reports `warn_severity`. | `Scan (gitleaks, bearer)` |
| Trivy | Terraform/OpenTofu misconfiguration. Opt-in via `iac_scan`. | `IaC Scan (trivy)` |

Gitleaks and Bearer share one job because GitHub bills each job for at least one minute and both finish in seconds. A Gitleaks finding fails the job before Bearer runs (`security-scan.yml:54-58`).

```yaml
name: Security

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read
  pull-requests: read   # required: gitleaks lists PR commits on pull_request events

jobs:
  security:
    uses: domengabrovsek/github-actions/.github/workflows/security-scan.yml@main
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `fail_severity` | `critical,high` | Bearer severities that fail the build. |
| `warn_severity` | `medium,low,warning` | Bearer severities reported without failing. |
| `iac_scan` | `false` | Run the Trivy IaC config scan and publish its SARIF to code scanning. |
| `iac_path` | `terraform` | Directory scanned when `iac_scan` is enabled. |

## Notes

- The caller **must** grant `pull-requests: read` at the workflow level. A reusable workflow's token cannot exceed the caller's grant, so a caller that sets only `contents: read` makes gitleaks 403 when listing PR commits and the secret scan fails without ever scanning.
- Enabling `iac_scan` additionally requires `security-events: write` at the caller's workflow level, otherwise the SARIF upload is rejected. Trivy itself runs with `continue-on-error`, so a scanner fault never blocks a PR.

## Scanning IaC

```yaml
permissions:
  contents: read
  pull-requests: read
  security-events: write

jobs:
  security:
    uses: domengabrovsek/github-actions/.github/workflows/security-scan.yml@main
    with:
      iac_scan: true
      iac_path: terraform
```
