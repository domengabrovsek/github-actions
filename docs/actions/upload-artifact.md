# `upload-artifact`

Thin wrapper around [`actions/upload-artifact`](https://github.com/actions/upload-artifact), SHA-pinned centrally so every repo tracks the same version via `@main`. Uploads files as a workflow artifact, e.g. to hand a build to a later job.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/upload-artifact@main
    with:
      name: dist
      path: dist/
      retention-days: 1
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `path` | required | File, directory, or wildcard pattern to upload. |
| `name` | `artifact` | Artifact name. |
| `if-no-files-found` | `warn` | Behavior when nothing matches `path` (`warn`, `error`, `ignore`). |
| `retention-days` | repo default | Days to keep the artifact. |
| `compression-level` | `6` | Zip compression, 0 (none) to 9 (best). |
| `overwrite` | `false` | Delete an existing artifact of the same name first. |
| `include-hidden-files` | `false` | Include dotfiles. |

## Outputs

| Output | Description |
|--------|-------------|
| `artifact-id` | GitHub ID of the uploaded artifact. |
| `artifact-url` | Download URL in the run UI. |
| `artifact-digest` | SHA-256 digest of the artifact. |

## Notes

- Pins `upload-artifact` to a single SHA in [`.github/actions/upload-artifact/action.yml`](../../.github/actions/upload-artifact/action.yml).
- Pinned to v7. Repos on v4 should test on adoption - artifact v5+ carried breaking changes (immutable artifacts, no re-upload to the same name).
