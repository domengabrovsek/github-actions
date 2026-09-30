# Telegram notifications

Telegram notifications for the full PR lifecycle, plus CI, deploy and terraform events. A router workflow dispatches PR events to per-event handlers, and one formatter renders their messages so they all look the same.

## Setup

Add two repository variables (Settings -> Secrets and variables -> Actions -> Variables):

- `TELEGRAM_API_URL` - webhook URL for your Telegram bot API (e.g. `https://abc123.lambda-url.eu-central-1.on.aws/`).
- `TELEGRAM_CHAT_ID` - chat ID messages are sent to (e.g. `123456789`).

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

**Fork PRs:** use `pull_request_target` so `vars.*` resolves for forks. `pull_request_target` runs with the base repo's token and variables, so running fork code there would hand them to the fork author. This is only safe because the router chain never runs `actions/checkout` and never executes PR-head code - it only reads event-payload metadata. Do not add checkout to this chain.

## Events

The router (`notify.yml`) dispatches PR events to the first eight handlers below. It skips comments and reviews written by bots (`notify.yml:86-104`). `ci-status.yml` is not routed; it needs its own trigger (see [CI status](#ci-status)). Each handler also works on its own if you only want one notification.

| Handler | Event | Header emoji |
|---------|-------|-------|
| `pr-opened.yml` | PR opened | 🚀 |
| `pr-updated.yml` | New commits pushed to PR | 🔄 |
| `pr-merged.yml` | PR merged | ✅ |
| `pr-closed.yml` | PR closed without merge | ❌ |
| `pr-commented.yml` | Comment on PR | 💬 |
| `pr-review-comment.yml` | Inline code review comment | 🔍 |
| `pr-review.yml` | Review submitted | ✅ approved, 🔴 changes requested, 💬 commented |
| `pr-review-requested.yml` | Review requested | 👋 |
| `ci-status.yml` | Watched workflow completed | ✅ success, ❌ failure, ⚠️ cancelled, ⏰ timed out, ⏭️ skipped |

All handlers take the same two inputs, `api_url` and `chat_id`.

### CI status

`ci-status.yml` uses `workflow_run`, which must name the workflows to watch, so it needs its own trigger file:

```yaml
name: CI Notifications

on:
  workflow_run:
    workflows: ["CI", "Tests"]   # the `name:` of each workflow to watch
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

## Formatter (`telegram-notify.yml` and the `notify` action)

The message builder lives in the composite action `.github/actions/notify`. It owns all formatting: header emoji, labels, field order, spacing, 300-char body truncation, and the derived Repository line. Callers supply data through typed inputs and pick the layout with `event_type`. There is no free-form message input. Free text such as a comment goes in `body`, which renders on a fixed Comment line, so every message it sends looks the same. It fails the step when the webhook returns a non-2xx status.

The builder has two entry points, and both render the same message:

| Entry point | Use it when | Cost |
| --- | --- | --- |
| `telegram-notify.yml` reusable workflow | You need a whole job, for example as a handler or after other jobs finish | One extra job, billed at least one minute |
| `domengabrovsek/github-actions/.github/actions/notify@main` step | A job that already runs can send the message as its last step | No extra job |

Both take the same inputs. Mind the names: the `telegram-notify.yml` workflow wraps the `notify` action, not the `telegram-notify` action described [below](#caller-titled-sender-telegram-notify-action). The workflow is a one-job wrapper around the action (`telegram-notify.yml:92-99`). Formatting and sending share that one job, because GitHub bills every job for at least one minute.

The PR and CI handlers above call the workflow for you. Call it directly for deploy and terraform events, which have no dedicated handler. This example runs on `push`, the only event that sets `github.event.head_commit`:

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

The same message sent as the last step of a deploy job that already runs. The action fails its step on a non-2xx webhook response, so `continue-on-error: true` keeps a Telegram outage from marking a good deploy as failed:

```yaml
      - if: always()
        continue-on-error: true
        uses: domengabrovsek/github-actions/.github/actions/notify@main
        with:
          api_url: ${{ vars.TELEGRAM_API_URL }}
          chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
          event_type: ${{ job.status == 'success' && 'deploy_success' || 'deploy_failure' }}
          branch_head: ${{ github.ref_name }}
          actor: ${{ github.actor }}
          commit: ${{ github.event.head_commit.message }}
          link: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

**`event_type` values:** `pr_opened`, `pr_updated`, `pr_merged`, `pr_closed`, `pr_review_requested`, `pr_commented`, `pr_review_comment`, `pr_review`, `ci_status`, `deploy_started`, `deploy_success`, `deploy_failure`, `terraform_started`, `terraform_result`, `drift`.

**Data inputs** (all optional; the formatter renders the subset relevant to the event and omits empties): `status`, `title`, `actor`, `reviewer`, `branch_head`, `branch_base`, `file`, `body`, `commits`, `trigger`, `commit`, `stacks`, `link`, `site`.

`trigger` takes a raw event name and renders a label: `workflow_dispatch` shows as Manual, and `schedule`, `push`, `pull_request` and `release` as Automatic (scheduled), (push), (pull request) and (release). An event outside that map shows as Automatic (`<event>`) (`.github/actions/notify/action.yml:113-122`).

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

## Caller-titled sender (`telegram-notify` action)

`.github/actions/telegram-notify` is a separate composite action that does not use the formatter. The caller passes the header line as `title`, and the action appends fields read from the run's `github` context:

| Input | Default | Description |
|-------|---------|-------------|
| `title` | required | First line of the message, including any emoji. |
| `api_url` | required | Telegram API webhook URL. |
| `chat_id` | required | Chat ID to send to. |
| `site_url` | `''` | Adds a Site line when set. |
| `show_branch` | `true` | Adds the ref name. |
| `show_author` | `true` | Adds `github.actor`. |
| `show_commit` | `true` | Adds the short SHA and the head commit subject. |

Every message also gets the Repository and Workflow run links. The send ignores errors (`|| true`), so a failed send never fails the job (`.github/actions/telegram-notify/action.yml:79-84`). Because the caller writes the header, its messages can differ from the formatter's layout. New callers use the `notify` action, which keeps every message in one layout.

