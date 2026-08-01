# github-actions 🤖

Reusable GitHub Actions shared across my personal repos: Telegram notifications 📱 plus the common CI, build, and deploy steps 🏗️. Update them here once and every repo that references this one picks up the change.

- [CI, Build & Deploy](#ci-build--deploy-)
- [Notifications](#workflows-)

## Quick Start 🚀

The simplest way to get all PR notifications. Add this workflow to your repo:

```yaml
name: Notifications

on:
  pull_request:
    types: [opened, closed, synchronize, review_requested]
    branches: [main, master]
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

One job. Two params. You get notifications for all PR lifecycle events:

| Workflow | Event | Emoji |
|----------|-------|-------|
| `pr-opened.yml` | PR opened | 🚀 |
| `pr-updated.yml` | New commits pushed to PR | 🔄 |
| `pr-merged.yml` | PR merged | ✅ |
| `pr-closed.yml` | PR closed without merge | ❌ |
| `pr-commented.yml` | Comment on PR | 💬 |
| `pr-review-comment.yml` | Inline code review comment | 🔍 |
| `pr-review.yml` | Review submitted (approved / changes requested) | 👀 |
| `pr-review-requested.yml` | Review requested | 👋 |
| `ci-status.yml` | CI workflow completed (success / failure) | ✅❌⚠️ |

## Setup 🔧

**Option 1: Use Repository Variables (Recommended)**

Add these variables to your repository (Settings → Secrets and variables → Actions → Variables):

- `TELEGRAM_API_URL` - The webhook URL for your Telegram bot API (e.g., `https://abc123.execute-api.eu-central-1.amazonaws.com/prod/webhook`)
- `TELEGRAM_CHAT_ID` - Telegram chat ID where messages will be sent (e.g., `1542727970`)

**Option 2: Hardcode in Workflow**

Directly specify `api_url` and `chat_id` in your workflow file (useful for public repos or different chat IDs per repo).

## CI, Build & Deploy 🏗️

Reusable CI steps extracted from the repos that share them. Pin with `@main` (or a tag / SHA once you want to freeze a version), and update here to roll changes out everywhere.

### Setup Node + npm (composite action) ⚙️

Checkout + `setup-node` (from `.nvmrc` by default) + `npm ci` with caching, in one step. Use it inside your own jobs when `node-ci.yml` is not a fit.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/setup-node-npm@main
  - run: npm run build
```

Inputs: `node-version-file` (default `.nvmrc`), `node-version` (overrides the file), `install` (default `true`), `install-args`, `fetch-depth` (default `1`).

The baseline install is `npm ci --ignore-scripts --no-audit --no-fund` (supply-chain safety + less CI noise), applied by both the composite and `node-ci.yml`. `install-args` is appended for repo-specific extras.

### Node CI 🧪

Runs lint / format / typecheck / test / build as separate jobs plus a `Gate` aggregator for branch protection. Each check runs only when you pass its command, so a repo enables exactly what it has scripts for.

```yaml
name: Pull Request

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  ci:
    uses: domengabrovsek/github-actions/.github/workflows/node-ci.yml@main
    with:
      typecheck_command: npm run typecheck
      test_command: npm run test
      build_artifact_path: dist
```

Inputs: `lint_command` (default `npm run lint`), `format_command`, `typecheck_command`, `test_command`, `build_command` (default `npm run build`), `build_artifact_path` (asserted to exist after build), `actionlint` (boolean, default `false` - lints `.github/workflows` when `true`), `node-version-file`, `install-args`, `runs-on`. Leave a `*_command` empty to skip that check.

**Service containers:** tests that need Postgres or another service keep their own job in the consuming repo. A `services:` map cannot be passed through workflow inputs. Point `test_command` at unit tests only, or leave it empty.

### Security Scan 🔒

Gitleaks (secret detection over full history) + Bearer (SAST, fails on high severity, reports the rest).

```yaml
name: Security

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  security:
    uses: domengabrovsek/github-actions/.github/workflows/security-scan.yml@main
```

Inputs: `fail_severity` (default `critical,high`), `warn_severity` (default `medium,low,warning`).

### Cloudflare Pages Deploy ☁️

Builds a static site and deploys it to Cloudflare Pages with wrangler. The Cloudflare token is read at runtime from AWS SSM via OIDC, so no long-lived token sits in repo secrets.

```yaml
name: Deploy

on:
  push:
    branches: [main]

permissions:
  id-token: write   # required for AWS OIDC
  contents: read

jobs:
  deploy:
    uses: domengabrovsek/github-actions/.github/workflows/cloudflare-pages-deploy.yml@main
    with:
      project_name: my-pages-project
      cloudflare_account_id: ${{ vars.CLOUDFLARE_ACCOUNT_ID }}
      aws_role_arn: ${{ vars.AWS_DEPLOY_ROLE_ARN }}
```

Inputs: `project_name`, `cloudflare_account_id`, `aws_role_arn` (required); `aws_region` (default `eu-central-1`), `ssm_token_path` (default `/home-infra/cloudflare/pages_deploy_token`), `build_command` (default `npm run build`), `output_directory` (default `dist`), `branch`, `node-version-file`.

## Workflows 🚀

### Notification Router (Recommended) 🎯

Routes all PR events to the correct notification handler automatically. See [Quick Start](#quick-start-) above.

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/notify.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

### Individual Workflows

You can also use individual workflows if you only want specific notifications:

#### Telegram Notify (central formatter) 💬

The single workflow that renders every Telegram message. Callers supply data
through typed inputs and pick the layout with `event_type`; the workflow owns all
formatting (emoji, labels, order, spacing, 300-char body truncation, and the
derived Repository line). There is no free-form `message` input by design, so
every notification looks the same.

The PR/CI handlers below call it for you. Call it directly for deploy and
terraform events, which have no dedicated handler:

```yaml
jobs:
  notify-start:
    uses: domengabrovsek/github-actions/.github/workflows/telegram-notify.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
      event_type: deploy_started
      branch_head: ${{ github.ref_name }}
      actor: ${{ github.actor }}
      commit: ${{ github.event.head_commit.message }}
      link: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

`event_type` values: `pr_opened`, `pr_updated`, `pr_merged`, `pr_closed`,
`pr_review_requested`, `pr_commented`, `pr_review_comment`, `pr_review`,
`ci_status`, `deploy_started`, `deploy_success`, `deploy_failure`,
`terraform_started`, `terraform_result`, `drift`.

Data inputs (all optional; the formatter renders the subset relevant to the
event and omits empties): `status`, `title`, `actor`, `reviewer`, `branch_head`,
`branch_base`, `file`, `body`, `commits`, `trigger`, `commit`, `stacks`, `link`.
`status` drives the emoji for the dynamic families - `pr_review`
(`approved` / `changes_requested` / `commented`), `ci_status`
(`success` / `failure` / `cancelled` / `timed_out` / `skipped`),
`terraform_result` (`success` / `failure`), `drift` (`clean` / `detected`).

#### PR Opened Notification 🎉

Sends a notification when a PR is opened, including commit details.

```yaml
jobs:
  pr-opened:
    uses: domengabrovsek/github-actions/.github/workflows/pr-opened.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Updated Notification 🔄

Sends a notification when changes are pushed to a PR, including new commits.

```yaml
jobs:
  pr-updated:
    uses: domengabrovsek/github-actions/.github/workflows/pr-updated.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Merged Notification ✅

Sends a notification when a PR is merged.

```yaml
jobs:
  pr-merged:
    uses: domengabrovsek/github-actions/.github/workflows/pr-merged.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Closed Notification ❌

Sends a notification when a PR is closed without being merged.

```yaml
jobs:
  pr-closed:
    uses: domengabrovsek/github-actions/.github/workflows/pr-closed.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Commented Notification 💬

Sends a notification when a comment is left on a PR.

```yaml
jobs:
  pr-commented:
    uses: domengabrovsek/github-actions/.github/workflows/pr-commented.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Review Comment Notification 🔍

Sends a notification when an inline code review comment is left on a PR.

```yaml
jobs:
  pr-review-comment:
    uses: domengabrovsek/github-actions/.github/workflows/pr-review-comment.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Review Notification 👀

Sends a notification when a PR review is submitted (approved, changes requested, or commented).

```yaml
jobs:
  pr-review:
    uses: domengabrovsek/github-actions/.github/workflows/pr-review.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Review Requested Notification 👋

Sends a notification when someone is requested to review a PR.

```yaml
jobs:
  pr-review-requested:
    uses: domengabrovsek/github-actions/.github/workflows/pr-review-requested.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### CI Status Notification ⚙️

Sends a notification when a workflow run completes (success, failure, cancelled).

**Note:** `workflow_run` requires specifying which workflows to watch, so this needs its own trigger file:

```yaml
name: CI Notifications

on:
  workflow_run:
    workflows: ["CI", "Tests"]  # Customize: names of workflows to watch
    types: [completed]

permissions:
  contents: read

jobs:
  ci-status:
    uses: domengabrovsek/github-actions/.github/workflows/ci-status.yml@main
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

## Different Chat IDs per Repository 🎯

You can send notifications to different Telegram chats for different repositories:

**Repository A (sends to chat 123456789):**

```yaml
# Set TELEGRAM_CHAT_ID variable to "123456789" in repo settings
uses: domengabrovsek/github-actions/.github/workflows/notify.yml@main
with:
  api_url: ${{ vars.TELEGRAM_API_URL }}
  chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

**Repository B (sends to chat 987654321):**

```yaml
# Set TELEGRAM_CHAT_ID variable to "987654321" in repo settings
uses: domengabrovsek/github-actions/.github/workflows/notify.yml@main
with:
  api_url: ${{ vars.TELEGRAM_API_URL }}
  chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```
