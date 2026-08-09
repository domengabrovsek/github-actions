# `setup-opentofu`

Thin wrapper around [`opentofu/setup-opentofu`](https://github.com/opentofu/setup-opentofu), SHA-pinned centrally so every repo tracks the same version via `@main`. Installs OpenTofu and puts `tofu` on the `PATH`.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/setup-opentofu@main
    with:
      tofu_version: 1.9.0
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `tofu_version` | `latest` | OpenTofu version to install (e.g. `1.9.0`). |
| `tofu_wrapper` | `true` | Wrap the `tofu` binary to expose stdout/stderr/exitcode as step outputs. Set `false` to call `tofu` directly. |
| `github_token` | workflow token | Token used to look up the OpenTofu release. |
| `cli_config_credentials_hostname` | `''` | Registry hostname whose credentials are written to the CLI config. |
| `cli_config_credentials_token` | `''` | API token written to the CLI config for that hostname. |

## Notes

- Pins `setup-opentofu` to a single SHA in [`.github/actions/setup-opentofu/action.yml`](../../.github/actions/setup-opentofu/action.yml).
