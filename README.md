# github-actions 🤖

GitHub Actions that I reuse in other repos to send messages regarding updates to my Telegram bot. 📱

## Setup 🔧

**Option 1: Use Repository Variables (Recommended)**


Add these variables to your repository (Settings → Secrets and variables → Actions → Variables):

- `TELEGRAM_API_URL` - The webhook URL for your Telegram bot API (e.g., `https://abc123.execute-api.eu-central-1.amazonaws.com/prod/webhook`)
- `TELEGRAM_CHAT_ID` - Telegram chat ID where messages will be sent (e.g., `1542727970`)

**Option 2: Hardcode in Workflow**

Directly specify `api_url` and `chat_id` in your workflow file (useful for public repos or different chat IDs per repo).

## Workflows 🚀

### Send Telegram Message 💬

Sends a custom message to Telegram.

**Using Repository Variables:**

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
      message: "Your message here"
```

**Using Hardcoded Values:**

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      api_url: "https://your-api.execute-api.region.amazonaws.com/prod/webhook"
      chat_id: "123456789"
      message: "Your message here"
```

**Using Secrets (for sensitive chat IDs):**

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      api_url: ${{ secrets.TELEGRAM_API_URL }}
      chat_id: ${{ secrets.TELEGRAM_CHAT_ID }}
      message: "Your message here"
```

### PR Opened Notification 🎉

Sends a notification when a PR is opened.

```yaml
jobs:
  pr-opened:
    uses: domengabrovsek/github-actions/.github/workflows/pr-opened.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

### PR Updated Notification 🔄

Sends a notification when changes are pushed to a PR.

```yaml
jobs:
  pr-updated:
    uses: domengabrovsek/github-actions/.github/workflows/pr-updated.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

### PR Merged Notification ✅

Sends a notification when a PR is merged.

```yaml
jobs:
  pr-merged:
    uses: domengabrovsek/github-actions/.github/workflows/pr-merged.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
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
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}

  pr-updated:
    if: github.event.action == 'synchronize'
    uses: domengabrovsek/github-actions/.github/workflows/pr-updated.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}

  pr-merged:
    if: github.event.action == 'closed' && github.event.pull_request.merged == true
    uses: domengabrovsek/github-actions/.github/workflows/pr-merged.yml@master
    with:
      api_url: ${{ vars.TELEGRAM_API_URL }}
      chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

## Different Chat IDs per Repository 🎯

You can send notifications to different Telegram chats for different repositories:

**Repository A (sends to chat 123456789):**

```yaml
# Set TELEGRAM_CHAT_ID variable to "123456789" in repo settings
uses: domengabrovsek/github-actions/.github/workflows/pr-opened.yml@master
with:
  api_url: ${{ vars.TELEGRAM_API_URL }}
  chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```

**Repository B (sends to chat 987654321):**

```yaml
# Set TELEGRAM_CHAT_ID variable to "987654321" in repo settings
uses: domengabrovsek/github-actions/.github/workflows/pr-opened.yml@master
with:
  api_url: ${{ vars.TELEGRAM_API_URL }}
  chat_id: ${{ vars.TELEGRAM_CHAT_ID }}
```
