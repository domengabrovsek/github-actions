# github-actions

Reusable GitHub Actions workflows.

## Send Telegram Message

Sends a message to Telegram.

### Usage

```yaml
jobs:
  notify:
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      message: "Your message here"
```

### Setup

Add `TELEGRAM_API_URL` secret to this repository (Settings → Secrets → Actions).

### Example: PR Notifications

```yaml
name: Notifications

on:
  pull_request:
    types: [opened, closed]

jobs:
  pr-opened:
    if: github.event.action == 'opened'
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      message: "🚀 New PR: ${{ github.event.pull_request.title }} by ${{ github.event.pull_request.user.login }}"

  pr-merged:
    if: github.event.action == 'closed' && github.event.pull_request.merged == true
    uses: domengabrovsek/github-actions/.github/workflows/send-telegram-message.yml@master
    with:
      message: "✅ PR Merged: ${{ github.event.pull_request.title }}"
```
