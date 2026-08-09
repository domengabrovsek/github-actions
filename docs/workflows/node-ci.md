# `node-ci.yml`

Runs lint / format / typecheck / test / build as separate jobs plus a `Gate` aggregator for branch protection. Each check runs only when you pass its command, so a repo enables exactly what it has scripts for.

```yaml
name: Pull Request

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  ci:
    uses: domengabrovsek/github-actions/.github/workflows/node-ci.yml@main
    with:
      typecheck_command: npm run typecheck
      test_command: npm run test
      build_artifact_path: dist
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `lint_command` | `npm run lint` | Empty string skips the lint job. |
| `format_command` | `''` | Format-check command. Empty skips. |
| `typecheck_command` | `''` | Typecheck command. Empty skips. |
| `test_command` | `''` | Test command. Empty skips. |
| `build_command` | `npm run build` | Empty skips the build job. |
| `build_artifact_path` | `''` | Path asserted to exist after build (e.g. `dist`). Empty skips the assertion. |
| `actionlint` | `false` | Lint `.github/workflows` with actionlint. Off by default. |
| `node-version-file` | `.nvmrc` | File that pins the Node version. |
| `install-args` | `''` | Extra args appended to the baseline `npm ci`. |
| `runs-on` | `ubuntu-latest` | Runner label(s) for the check jobs. |

## Notes

- **Gate:** the `Gate` job aggregates every check into one status. Point branch protection at `ci / Gate` (skipped checks count as passing).
- **Service containers:** tests that need Postgres or another service keep their own job in the consuming repo - a `services:` map cannot be passed through workflow inputs. Point `test_command` at unit tests only, or leave it empty.
- Each job installs via the [`setup-node-npm`](../actions/setup-node-npm.md) composite.
