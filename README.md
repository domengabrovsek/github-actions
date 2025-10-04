# github-actions 🤖

GitHub Actions that I reuse in other repos to send messages regarding updates to my Telegram bot. 📱

## Setup 🔧

Add `TELEGRAM_API_URL` secret to this repository (Settings → Secrets → Actions).

## Workflows 🚀

### Send Telegram Message 💬

Sends a custom message to Telegram.

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      message: "Your message here"
```

### PR Opened Notification 🎉

Sends a notification when a PR is opened.

```yaml
jobs:
  pr-opened:
    uses: domengabrovsek/github-actions/.github/workflows/pr-opened.yml@master
```

### PR Updated Notification 🔄

Sends a notification when changes are pushed to a PR.

```yaml
jobs:
  pr-updated:
    uses: domengabrovsek/github-actions/.github/workflows/pr-updated.yml@master
```

### PR Merged Notification ✅

Sends a notification when a PR is merged.

```yaml
jobs:
  pr-merged:
    uses: domengabrovsek/github-actions/.github/workflows/pr-merged.yml@master
```

## Example: Full PR Notifications 📋

```yaml
name: Notifications

on:
  pull_request:
    types: [opened, closed, synchronize]
    branches: [main, master]

jobs:
  pr-opened:
    if: github.event.action == 'opened'
    uses: domengabrovsek/github-actions/.github/workflows/pr-opened.yml@master

  pr-updated:
    if: github.event.action == 'synchronize'
    uses: domengabrovsek/github-actions/.github/workflows/pr-updated.yml@master

  pr-merged:
    if: github.event.action == 'closed' && github.event.pull_request.merged == true
    uses: domengabrovsek/github-actions/.github/workflows/pr-merged.yml@master
```
