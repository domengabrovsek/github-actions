# `github-app-token`

Thin wrapper around [`actions/create-github-app-token`](https://github.com/actions/create-github-app-token), SHA-pinned centrally so every repo tracks the same version via `@main`. Mints a short-lived installation token, so a job acts as the App (`<app-slug>[bot]`) instead of `github-actions[bot]`.

```yaml
steps:
  - id: token
    uses: domengabrovsek/github-actions/.github/actions/github-app-token@main
    with:
      client-id: ${{ vars.APP_CLIENT_ID }}
      private-key: ${{ secrets.APP_PRIVATE_KEY }}
      permission-pull-requests: write
  - run: gh pr comment "$PR" --body "Hello"
    env:
      GH_TOKEN: ${{ steps.token.outputs.token }}
      PR: ${{ github.event.pull_request.number }}
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `client-id` | required | GitHub App client ID. |
| `private-key` | required | GitHub App private key. |
| `owner` | repo owner | Owner of the App installation. |
| `repositories` | current repo | Repositories the token may access. |
| `permission-contents` | installation's | `read` or `write`. |
| `permission-pull-requests` | installation's | `read` or `write`. |

## Outputs

| Output | Description |
|--------|-------------|
| `token` | Installation access token, revoked when the job ends. |
| `app-slug` | App slug. The App posts as `<app-slug>[bot]`. |
| `installation-id` | Installation ID of the App. |

## Notes

- Pins `create-github-app-token` to a single SHA in [`.github/actions/github-app-token/action.yml`](../../.github/actions/github-app-token/action.yml).
- Request only the permissions a job needs. A token cannot exceed the App's installation permissions.
- Unlike `GITHUB_TOKEN`, events caused by an App token trigger other workflows.
