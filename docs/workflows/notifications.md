# Telegram notifications

Telegram notifications for the full PR lifecycle, plus deploy and terraform events. One router workflow dispatches events to per-event handlers, and a single formatter renders every message so they all look the same.

## Setup

Add two repository variables (Settings -> Secrets and variables -> Actions -> Variables):

- `TELEGRAM_API_URL` - webhook URL for your Telegram bot API (e.g. `https://abc123.lambda-url.eu-central-1.on.aws/`).
- `TELEGRAM_CHAT_ID` - chat ID messages are sent to (e.g. `1542727970`).

You can hardcode `api_url` / `chat_id` in the workflow instead - useful for public repos or per-repo chat IDs.

## Quick start (router)

The router handles every PR event with one job. Add this workflow to your repo:

```yaml
name: Notifications

on:
  pull_request:            # or pull_request_target, if the repo accepts fork PRs
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

**Fork PRs:** use `pull_request_target` so `vars.*` resolves for forks. This is only safe because the router chain never runs `actions/checkout` and never executes PR-head code - it only reads event-payload metadata. Do not add checkout to this chain.

## Events

The router (`notify.yml`) dispatches to these per-event handlers. Each is also usable on its own if you only want one notification.

| Handler | Event | Emoji |
|---------|-------|-------|
| `pr-opened.yml` | PR opened | 🚀 |
| `pr-updated.yml` | New commits pushed to PR | 🔄 |
| `pr-merged.yml` | PR merged | ✅ |
| `pr-closed.yml` | PR closed without merge | ❌ |
| `pr-commented.yml` | Comment on PR | 💬 |
| `pr-review-comment.yml` | Inline code review comment | 🔍 |
| `pr-review.yml` | Review submitted (approved / changes requested) | 👀 |
| `pr-review-requested.yml` | Review requested | 👋 |
| `ci-status.yml` | CI workflow completed (success / failure) | ✅❌⚠️ |

All handlers take the same two inputs, `api_url` and `chat_id`.

### CI status

`ci-status.yml` uses `workflow_run`, which must name the workflows to watch, so it needs its own trigger file:

```yaml
name: CI Notifications

on:
  workflow_run:
    workflows: ["CI", "Tests"]   # names of workflows to watch
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

## Formatter (`telegram-notify.yml`)

The single workflow that renders every Telegram message. Callers supply data through typed inputs and pick the layout with `event_type`; the workflow owns all formatting (emoji, labels, order, spacing, 300-char body truncation, derived Repository line). There is no free-form `message` input by design, so every notification looks the same.

The PR/CI handlers above call it for you. Call it directly for deploy and terraform events, which have no dedicated handler:

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

**`event_type` values:** `pr_opened`, `pr_updated`, `pr_merged`, `pr_closed`, `pr_review_requested`, `pr_commented`, `pr_review_comment`, `pr_review`, `ci_status`, `deploy_started`, `deploy_success`, `deploy_failure`, `terraform_started`, `terraform_result`, `drift`.

**Data inputs** (all optional; the formatter renders the subset relevant to the event and omits empties): `status`, `title`, `actor`, `reviewer`, `branch_head`, `branch_base`, `file`, `body`, `commits`, `trigger`, `commit`, `stacks`, `link`.

`status` drives the emoji for the dynamic families:

- `pr_review` - `approved` / `changes_requested` / `commented`
- `ci_status` - `success` / `failure` / `cancelled` / `timed_out` / `skipped`
- `terraform_result` - `success` / `failure`
- `drift` - `clean` / `detected`

## Per-repo chat IDs

Point different repos at different chats by setting each repo's `TELEGRAM_CHAT_ID` variable - the workflow reference stays identical:

```yaml
# Repo A sets TELEGRAM_CHAT_ID = 123456789, Repo B sets 987654321
uses: domengabrovsek/github-actions/.github/workflows/notify.yml@main
with:
  api_url: ${{ vars.TELEGRAM_API_URL }}
  chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```
