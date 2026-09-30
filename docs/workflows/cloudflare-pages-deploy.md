# `cloudflare-pages-deploy.yml`

Builds a static site and deploys it to Cloudflare Pages with wrangler. The Cloudflare token is read at runtime from AWS SSM via OIDC, so no long-lived token sits in repo secrets.

```yaml
name: Deploy

on:
  push:
    branches: [main]

permissions:
  id-token: write   # required for AWS OIDC
  contents: read

jobs:
  deploy:
    uses: domengabrovsek/github-actions/.github/workflows/cloudflare-pages-deploy.yml@main
    with:
      project_name: my-pages-project
      cloudflare_account_id: ${{ vars.CLOUDFLARE_ACCOUNT_ID }}
      aws_role_arn: ${{ vars.AWS_DEPLOY_ROLE_ARN }}
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `project_name` | required | Cloudflare Pages project name. |
| `cloudflare_account_id` | required | Cloudflare account ID. |
| `aws_role_arn` | required | AWS IAM role ARN to assume via OIDC for reading the deploy token. |
| `aws_region` | `eu-central-1` | AWS region for the SSM lookup. |
| `ssm_token_path` | `/home-infra/cloudflare/pages_deploy_token` | SSM path holding the Cloudflare deploy token. |
| `build_command` | `npm run build` | Command that produces the static output. |
| `output_directory` | `dist` | Directory wrangler uploads to Pages. |
| `branch` | `''` | Deploy branch passed to wrangler (blank lets wrangler infer production vs preview). |
| `node-version-file` | `.nvmrc` | File that pins the Node version. |

## Notes

- Installs via the [`setup-node-npm`](../actions/setup-node-npm.md) composite (hardened `npm ci`) and configures AWS through the [`aws-credentials`](../actions/aws-credentials.md) wrapper.
- The build runs before AWS credentials are configured, so build scripts never see the AWS session or the Cloudflare token.
- The AWS keys go only to the SSM read step, and the Cloudflare token, masked, goes only to the deploy step. Neither is exported to the job environment (`cloudflare-pages-deploy.yml`, steps `Configure AWS credentials` through `Deploy with wrangler`).
- The deploy runs wrangler at the version pinned in the workflow's `WRANGLER_VERSION` env, so a new wrangler release never runs with the token until it is bumped here.
