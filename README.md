# github-actions 🤖

Reusable GitHub Actions shared across my personal repos: composite build steps, reusable CI / deploy workflows, and Telegram notifications. Update a step once here and every repo that references it picks up the change.

New here? Start with [docs/architecture.md](docs/architecture.md): the parts, how they call each other, the design decisions, and how to change them safely.

## How it works

Reference anything here with `@main`:

```yaml
- uses: domengabrovsek/github-actions/.github/actions/checkout@main          # composite action
# or
uses: domengabrovsek/github-actions/.github/workflows/node-ci.yml@main        # reusable workflow
```

Every third-party action is pinned to a commit SHA. Common ones are pinned once, in their wrapper action, so a version bump is a single edit that rolls out to all consumers via `@main`. Pin to a tag or SHA instead when you want to freeze a version.

## Composite actions

Building blocks for your own jobs. See [`docs/actions/`](docs/actions).

| Action | What it does | Docs |
|--------|--------------|------|
| `setup-node-npm` | Checkout + setup-node (`.nvmrc`) + hardened `npm ci`, cached | [docs](docs/actions/setup-node-npm.md) |
| `checkout` | Centrally-pinned wrapper for `actions/checkout` | [docs](docs/actions/checkout.md) |
| `aws-credentials` | Centrally-pinned wrapper for `configure-aws-credentials` (OIDC or static keys) | [docs](docs/actions/aws-credentials.md) |
| `setup-opentofu` | Centrally-pinned wrapper for `opentofu/setup-opentofu` | [docs](docs/actions/setup-opentofu.md) |
| `setup-terraform` | Centrally-pinned wrapper for `hashicorp/setup-terraform` | [docs](docs/actions/setup-terraform.md) |
| `upload-artifact` | Centrally-pinned wrapper for `actions/upload-artifact` | [docs](docs/actions/upload-artifact.md) |
| `download-artifact` | Centrally-pinned wrapper for `actions/download-artifact` | [docs](docs/actions/download-artifact.md) |
| `markdownlint` | Centrally-pinned wrapper for `markdownlint-cli2-action` | [docs](docs/actions/markdownlint.md) |
| `claude-code` | Centrally-pinned wrapper for `anthropics/claude-code-action` | [docs](docs/actions/claude-code.md) |
| `notify` | Renders and sends the standard Telegram message as a step | [docs](docs/workflows/notifications.md) |
| `telegram-notify` | Sends a Telegram message with a caller-written header | [docs](docs/workflows/notifications.md) |

## Reusable workflows

Whole jobs you call from a consumer repo. See [`docs/workflows/`](docs/workflows).

| Workflow | What it does | Docs |
|----------|--------------|------|
| `node-ci.yml` | Lint / format / typecheck / test / build in one `Checks` job | [docs](docs/workflows/node-ci.md) |
| `security-scan.yml` | Gitleaks (secrets) + Bearer (SAST) + opt-in Trivy (IaC) | [docs](docs/workflows/security-scan.md) |
| `cloudflare-pages-deploy.yml` | Build + deploy a static site to Cloudflare Pages via wrangler | [docs](docs/workflows/cloudflare-pages-deploy.md) |
| `notify.yml` + handlers | Telegram notifications for the full PR lifecycle, CI, deploy and terraform events | [docs](docs/workflows/notifications.md) |
| `reviewer.yml` | Claude PR review with inline comments, thread replies, and approval | [docs](docs/workflows/reviewer.md) |

## Decisions

Architecture decision records live in [`docs/adr/`](docs/adr):

- [0001: Central formatter owns all Telegram message layout](docs/adr/0001-central-telegram-message-formatter.md)

Other design choices and their reasons are in [docs/architecture.md](docs/architecture.md#design-decisions).
