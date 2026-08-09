# `download-artifact`

Thin wrapper around [`actions/download-artifact`](https://github.com/actions/download-artifact), SHA-pinned centrally so every repo tracks the same version via `@main`. Downloads a workflow artifact, e.g. a build produced by an earlier job.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/download-artifact@main
    with:
      name: dist
      path: dist/
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `name` | all artifacts | Name of the artifact to download. Empty downloads every artifact from the run. |
| `path` | working dir | Destination directory. |
| `pattern` | `''` | Glob of artifact names to download when `name` is empty. |
| `merge-multiple` | `false` | Merge downloads into one directory instead of a subdir each. |
| `github-token` | `''` | Token for downloading from another repo or run. Empty stays within the current run. |
| `repository` | current repo | Repo to download from. Used only with `github-token`. |
| `run-id` | current run | Run ID to download from. Used only with `github-token`. |

## Outputs

| Output | Description |
|--------|-------------|
| `download-path` | Absolute path the artifacts were downloaded to. |

## Notes

- Pins `download-artifact` to a single SHA in [`.github/actions/download-artifact/action.yml`](../../.github/actions/download-artifact/action.yml).
- Pinned to v8. Repos on v4 should test on adoption - artifact v5+ carried breaking changes.
