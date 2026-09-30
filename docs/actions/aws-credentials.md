# `aws-credentials`

Thin wrapper around [`aws-actions/configure-aws-credentials`](https://github.com/aws-actions/configure-aws-credentials), SHA-pinned centrally so every repo tracks the same version via `@main`. Configures AWS credentials for later steps, via OIDC role assumption or static keys.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/aws-credentials@main
    with:
      role-to-assume: ${{ vars.AWS_DEPLOY_ROLE_ARN }}
      aws-region: eu-central-1
```

OIDC role assumption needs `permissions: id-token: write` on the job.

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `aws-region` | required | AWS region to configure. |
| `role-to-assume` | `''` | IAM role ARN to assume via OIDC. Leave blank when using static keys. |
| `role-session-name` | `''` | Session name for the assumed role. |
| `audience` | `sts.amazonaws.com` | OIDC audience for the web identity token. Leave unset unless your IAM OIDC provider expects a different audience. An empty value also falls back to `sts.amazonaws.com`. |
| `role-duration-seconds` | `''` | Lifetime of the assumed-role credentials, in seconds. |
| `aws-access-key-id` | `''` | Static access key ID, as an alternative to `role-to-assume`. |
| `aws-secret-access-key` | `''` | Static secret access key. |
| `aws-session-token` | `''` | Session token for temporary static credentials. |
| `output-env-credentials` | `true` | Export the credentials to every later step. Set `false` to keep them out of later steps. |
| `output-credentials` | `false` | Return the credentials as step outputs (`aws-access-key-id`, `aws-secret-access-key`, `aws-session-token`), for passing to one step only. |

## Outputs

Set only when `output-credentials` is `true`.

| Output | Description |
|--------|-------------|
| `aws-access-key-id` | Access key ID. |
| `aws-secret-access-key` | Secret access key. |
| `aws-session-token` | Session token. |

## Notes

- Pins `configure-aws-credentials` to a single SHA in [`.github/actions/aws-credentials/action.yml`](../../.github/actions/aws-credentials/action.yml).
- `audience`, `output-env-credentials` and `output-credentials` are coalesced to their defaults when a caller passes an empty string. An empty string would otherwise override the upstream default, and an empty audience makes STS reject the token (`action.yml:65-68`).
