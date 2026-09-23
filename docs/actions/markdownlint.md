# `markdownlint`

Thin wrapper around [`DavidAnson/markdownlint-cli2-action`](https://github.com/DavidAnson/markdownlint-cli2-action), SHA-pinned centrally so every repo tracks the same version via `@main`. Lints Markdown with the repo's own `.markdownlint*` config.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/checkout@main
  - uses: domengabrovsek/github-actions/.github/actions/markdownlint@main
    with:
      globs: "**/*.md"
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `globs` | `*.{md,markdown}` | Newline-delimited globs of files to lint. |
| `config` | none | Base configuration file. Empty uses the repo's `.markdownlint*` files. |
| `fix` | `false` | Fix supported issues in place. |

## Notes

- Pins `markdownlint-cli2-action` to a single SHA in [`.github/actions/markdownlint/action.yml`](../../.github/actions/markdownlint/action.yml).
- Needs a prior checkout step.
- Pinned to v24. Repos coming from v19 or older may see new failures from rule MD060 (table column style).
