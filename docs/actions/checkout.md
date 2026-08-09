# `checkout`

Thin wrapper around [`actions/checkout`](https://github.com/actions/checkout), SHA-pinned centrally so every repo tracks the same version via `@main`. Drop-in replacement: inputs pass straight through and keep checkout's own defaults.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/checkout@main
    with:
      fetch-depth: 0
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `repository` | current repo | Repository name with owner. |
| `ref` | trigger ref | Branch, tag, or SHA to check out. |
| `token` | workflow token | Token used to fetch the repository. |
| `ssh-key` | `''` | SSH key, as an alternative to `token`. |
| `fetch-depth` | `1` | Commits to fetch. `0` fetches full history. |
| `fetch-tags` | `false` | Fetch tags even when `fetch-depth > 0`. |
| `submodules` | `false` | Check out submodules (`true`, `recursive`, or `false`). |
| `lfs` | `false` | Download Git LFS files. |
| `path` | workspace root | Directory under `$GITHUB_WORKSPACE` to check out into. |
| `clean` | `true` | Run `git clean` and reset before fetching. |
| `persist-credentials` | `true` | Keep the auth token in git config for later steps. |
| `sparse-checkout` | `''` | Newline-separated patterns for a sparse checkout. |

## Notes

- Pins `actions/checkout` to a single SHA. Bump it in [`.github/actions/checkout/action.yml`](../../.github/actions/checkout/action.yml) and every consumer picks it up via `@main`.
