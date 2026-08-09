# `security-scan.yml`

Bundles the two scans copy-pasted across repos:

- **Gitleaks** - secret detection over full history.
- **Bearer** - SAST, fails on high-severity findings and reports the rest.

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

## Notes

- The caller **must** grant `pull-requests: read` at the workflow level. A reusable workflow's token cannot exceed the caller's grant, so a caller that sets only `contents: read` makes gitleaks 403 when listing PR commits and the secret scan fails without ever scanning.
