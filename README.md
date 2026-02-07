# github-actions 🤖

GitHub Actions that I reuse in other repos to send messages regarding updates to my Telegram bot. 📱

## Quick Start 🚀

The simplest way to get all PR notifications. Add this workflow to your repo:

```yaml
name: Notifications

on:
  pull_request:
    types: [opened, closed, synchronize, labeled, review_requested]
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
    uses: domengabrovsek/github-actions/.github/workflows/notify.yml@master
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
| `pr-labeled.yml` | Label added to PR | 🏷️ |
| `ci-status.yml` | CI workflow completed (success / failure) | ✅❌⚠️ |

## Setup 🔧

**Option 1: Use Repository Variables (Recommended)**

Add these variables to your repository (Settings → Secrets and variables → Actions → Variables):

- `TELEGRAM_API_URL` - The webhook URL for your Telegram bot API (e.g., `https://abc123.execute-api.eu-central-1.amazonaws.com/prod/webhook`)
- `TELEGRAM_CHAT_ID` - Telegram chat ID where messages will be sent (e.g., `1542727970`)

**Option 2: Hardcode in Workflow**

Directly specify `api_url` and `chat_id` in your workflow file (useful for public repos or different chat IDs per repo).

## Workflows 🚀

### Notification Router (Recommended) 🎯

Routes all PR events to the correct notification handler automatically. See [Quick Start](#quick-start-) above.

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/notify.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

### Individual Workflows

You can also use individual workflows if you only want specific notifications:

#### Send Telegram Message 💬

Sends a custom message to Telegram.

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
      message: "Your message here"
```

#### PR Opened Notification 🎉

Sends a notification when a PR is opened, including commit details.

```yaml
jobs:
  pr-opened:
    uses: domengabrovsek/github-actions/.github/workflows/pr-opened.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Updated Notification 🔄

Sends a notification when changes are pushed to a PR, including new commits.

```yaml
jobs:
  pr-updated:
    uses: domengabrovsek/github-actions/.github/workflows/pr-updated.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Merged Notification ✅

Sends a notification when a PR is merged.

```yaml
jobs:
  pr-merged:
    uses: domengabrovsek/github-actions/.github/workflows/pr-merged.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Closed Notification ❌

Sends a notification when a PR is closed without being merged.

```yaml
jobs:
  pr-closed:
    uses: domengabrovsek/github-actions/.github/workflows/pr-closed.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Commented Notification 💬

Sends a notification when a comment is left on a PR.

```yaml
jobs:
  pr-commented:
    uses: domengabrovsek/github-actions/.github/workflows/pr-commented.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Review Comment Notification 🔍

Sends a notification when an inline code review comment is left on a PR.

```yaml
jobs:
  pr-review-comment:
    uses: domengabrovsek/github-actions/.github/workflows/pr-review-comment.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Review Notification 👀

Sends a notification when a PR review is submitted (approved, changes requested, or commented).

```yaml
jobs:
  pr-review:
    uses: domengabrovsek/github-actions/.github/workflows/pr-review.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Review Requested Notification 👋

Sends a notification when someone is requested to review a PR.

```yaml
jobs:
  pr-review-requested:
    uses: domengabrovsek/github-actions/.github/workflows/pr-review-requested.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

#### PR Labeled Notification 🏷️

Sends a notification when a label is added to a PR.

```yaml
jobs:
  pr-labeled:
    uses: domengabrovsek/github-actions/.github/workflows/pr-labeled.yml@master
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
    uses: domengabrovsek/github-actions/.github/workflows/ci-status.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

## Different Chat IDs per Repository 🎯

You can send notifications to different Telegram chats for different repositories:

**Repository A (sends to chat 123456789):**

```yaml
# Set TELEGRAM_CHAT_ID variable to "123456789" in repo settings
uses: domengabrovsek/github-actions/.github/workflows/notify.yml@master
with:
  api_url: ${{ vars.TELEGRAM_API_URL }}
  chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

**Repository B (sends to chat 987654321):**

```yaml
# Set TELEGRAM_CHAT_ID variable to "987654321" in repo settings
uses: domengabrovsek/github-actions/.github/workflows/notify.yml@master
with:
  api_url: ${{ vars.TELEGRAM_API_URL }}
  chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```
