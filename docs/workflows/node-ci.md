# `node-ci.yml`

Runs lint, format, typecheck, test and build as steps in one `Checks` job. Each check runs only when its command input is non-empty. Lint and build are on by default (`npm run lint`, `npm run build`); set their input to `''` to turn them off. Format, typecheck and test are off until you pass a command.

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
| `lint_command` | `npm run lint` | Lint command. Empty skips it. |
| `format_command` | `''` | Format-check command. Empty skips. |
| `typecheck_command` | `''` | Typecheck command. Empty skips. |
| `test_command` | `''` | Test command. Empty skips. |
| `build_command` | `npm run build` | Build command. Empty skips it and the artifact assertion. |
| `build_artifact_path` | `''` | Path asserted to exist after build (e.g. `dist`). Empty skips the assertion. |
| `actionlint` | `false` | Lint `.github/workflows` with actionlint. Off by default. |
| `node-version-file` | `.nvmrc` | File that pins the Node version. |
| `install-args` | `''` | Extra args appended to the baseline `npm ci --ignore-scripts --no-audit --no-fund`. |
| `runs-on` | `ubuntu-latest` | Runner label(s) for the `Checks` and `Actionlint` jobs. |

## Notes

- **One job:** all checks share one job because GitHub bills every job for at least one minute, and a job per check would repeat `npm ci`. Each check step uses `continue-on-error`, so every check runs and shows in the log. A final `Verify` step fails the job when any check failed. Skipped checks count as passing.
- **Status checks:** the job reports as `<caller job id> / Checks`, for example `ci / Checks`, plus `ci / Actionlint` when `actionlint` is on. This workflow has no gate job. Branch protection on managed repos requires a check named `Gate` (see [architecture](../architecture.md#design-decisions)), so the consumer adds this job under `jobs:` next to `ci`. It reports as `Gate` and fails when any job it lists did not succeed:

  ```yaml
    gate:
      name: Gate
      if: always()
      needs: [ci]           # add any other required jobs, such as a service-container test job
      runs-on: ubuntu-latest
      steps:
        - env:
            RESULTS: ${{ join(needs.*.result, ' ') }}
          run: for r in $RESULTS; do [ "$r" = success ] || exit 1; done
  ```

- **Service containers:** tests that need Postgres or another service keep their own job in the consuming repo - a `services:` map cannot be passed through workflow inputs. Point `test_command` at unit tests only, or leave it empty, and list the service-container job in the `Gate` job's `needs`.
- `Checks` installs once via the [`setup-node-npm`](../actions/setup-node-npm.md) composite, which sets up Node with npm caching. `--ignore-scripts` stops dependency install scripts from running. A caller cannot add steps inside this workflow, so a package that needs its install script gets it from the check command, such as `test_command: npm rebuild <pkg> && npm test`. `Actionlint` only checks out the repo.
