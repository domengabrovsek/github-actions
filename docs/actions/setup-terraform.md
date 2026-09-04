# `setup-terraform`

Thin wrapper around [`hashicorp/setup-terraform`](https://github.com/hashicorp/setup-terraform), SHA-pinned centrally so every repo tracks the same version via `@main`. Installs Terraform and puts `terraform` on the `PATH`.

For OpenTofu, use [`setup-opentofu`](setup-opentofu.md) instead.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/setup-terraform@main
    with:
      terraform_version: 1.15.5
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `terraform_version` | `latest` | Terraform version to install (e.g. `1.15.5`). |
| `terraform_wrapper` | `true` | Wrap the `terraform` binary to expose stdout/stderr/exitcode as step outputs. Set `false` to call `terraform` directly. |
| `cli_config_credentials_hostname` | `''` | Registry hostname whose credentials are written to the CLI config. |
| `cli_config_credentials_token` | `''` | API token written to the CLI config for that hostname. |

## Notes

- Pins `setup-terraform` to a single SHA in [`.github/actions/setup-terraform/action.yml`](../../.github/actions/setup-terraform/action.yml).
- The credentials block is written only when hostname and token are both set, so the empty defaults are a no-op.
- The upstream action runs on `node24`, which needs GitHub Actions runner v2.327.1 or later. Self-hosted runners below that will fail to start the step.
