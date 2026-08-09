# github-actions 🤖

Reusable GitHub Actions shared across my personal repos: composite build steps, reusable CI / deploy workflows, and Telegram notifications. Update a step once here and every repo that references it picks up the change.

## How it works

Reference anything here with `@main`:

```yaml
- uses: domengabrovsek/github-actions/.github/actions/checkout@main          # composite action
# or
uses: domengabrovsek/github-actions/.github/workflows/node-ci.yml@main        # reusable workflow
```

Every third-party action is SHA-pinned in exactly one place here, so a version bump is a single edit that rolls out to all consumers via `@main`. Pin to a tag or SHA instead when you want to freeze a version.

## Composite actions

Building blocks for your own jobs. See [`docs/actions/`](docs/actions).

| Action | What it does | Docs |
|--------|--------------|------|
| `setup-node-npm` | Checkout + setup-node (`.nvmrc`) + hardened `npm ci`, cached | [docs](docs/actions/setup-node-npm.md) |
| `checkout` | Centrally-pinned wrapper for `actions/checkout` | [docs](docs/actions/checkout.md) |
| `aws-credentials` | Centrally-pinned wrapper for `configure-aws-credentials` (OIDC or static keys) | [docs](docs/actions/aws-credentials.md) |
| `setup-opentofu` | Centrally-pinned wrapper for `opentofu/setup-opentofu` | [docs](docs/actions/setup-opentofu.md) |
| `upload-artifact` | Centrally-pinned wrapper for `actions/upload-artifact` | [docs](docs/actions/upload-artifact.md) |
| `download-artifact` | Centrally-pinned wrapper for `actions/download-artifact` | [docs](docs/actions/download-artifact.md) |

## Reusable workflows

Whole jobs you call from a consumer repo. See [`docs/workflows/`](docs/workflows).

| Workflow | What it does | Docs |
|----------|--------------|------|
| `node-ci.yml` | Lint / format / typecheck / test / build + a `Gate` for branch protection | [docs](docs/workflows/node-ci.md) |
| `security-scan.yml` | Gitleaks (secrets) + Bearer (SAST) | [docs](docs/workflows/security-scan.md) |
| `cloudflare-pages-deploy.yml` | Build + deploy a static site to Cloudflare Pages via wrangler | [docs](docs/workflows/cloudflare-pages-deploy.md) |
| `notify.yml` + handlers | Telegram notifications for the full PR lifecycle | [docs](docs/workflows/notifications.md) |

## Telegram notifications quick start

One job gives you notifications for every PR event. Set the `TELEGRAM_API_URL` and `TELEGRAM_CHAT_ID` repo variables, then:

```yaml
name: Notifications

on:
  pull_request:
    types: [opened, closed, synchronize, review_requested]
    branches: [main]
  issue_comment:
    types: [created]
  pull_request_review:
    types: [submitted]
  pull_request_review_comment:
    types: [created]

permissions:
  contents: read
  pull-requests: read

jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/notify.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

Full setup, event list, formatter reference, and per-repo chat IDs: [docs/workflows/notifications.md](docs/workflows/notifications.md).
