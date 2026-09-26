# `claude-code`

Thin wrapper around [`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action), SHA-pinned centrally so every repo tracks the same version via `@main`. Runs Claude Code in a job, authenticated with a Claude Pro/Max subscription or an API key.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/checkout@main
  - uses: domengabrovsek/github-actions/.github/actions/claude-code@main
    with:
      claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      github_token: ${{ secrets.GITHUB_TOKEN }}
      prompt: Review this pull request.
      claude_args: --model opus
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `prompt` | none | Instructions for Claude. Empty uses the triggering event's context. |
| `claude_args` | none | Extra Claude Code CLI arguments, such as `--model` or `--allowedTools`. |
| `claude_code_oauth_token` | none | Pro/Max OAuth token from `claude setup-token`. |
| `anthropic_api_key` | none | Anthropic API key, the alternative to the OAuth token. |
| `github_token` | none | GitHub token for comments. Empty uses the Claude GitHub App. |

## Outputs

| Output | Description |
|--------|-------------|
| `structured_output` | JSON string of the fields Claude returns when `claude_args` sets `--json-schema`. |

## Notes

- Pins `claude-code-action` to a single SHA in [`.github/actions/claude-code/action.yml`](../../.github/actions/claude-code/action.yml).
- Needs a prior checkout step.
- Setting `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB: "1"` in the job `env` needs `bubblewrap` on the runner. Install it first with `sudo apt-get install -y bubblewrap`.
