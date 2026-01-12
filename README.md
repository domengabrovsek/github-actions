# github-actions 🤖

GitHub Actions that I reuse in other repos to send messages regarding updates to my Telegram bot. 📱

## Setup 🔧

Add these secrets to your repository (Settings → Secrets → Actions):
- `TELEGRAM_API_URL` - The webhook URL for your Telegram bot API
- `TELEGRAM_CHAT_ID` - Default Telegram chat ID where messages will be sent

## Workflows 🚀

### Send Telegram Message 💬

Sends a custom message to Telegram.

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      message: "Your message here"
      # Optional: Override default chat_id from TELEGRAM_CHAT_ID secret
      # chat_id: "987654321"
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
