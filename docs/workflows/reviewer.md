# `reviewer.yml`

Claude reviews each pull request with inline comments, answers the owner's replies in its threads, and approves once no thread is open. The Claude token is read at runtime from AWS SSM via OIDC, so no repo stores it.

Save this as `.github/workflows/reviewer.yml` in the consumer repo:

```yaml
name: Reviewer

on:
  pull_request:
    types: [opened, reopened, ready_for_review, synchronize]
    branches: [main]
  pull_request_target:
    types: [labeled]
    branches: [main]
  pull_request_review_comment:
    types: [created]

permissions:
  contents: write
  pull-requests: write
  id-token: write

jobs:
  reviewer:
    if: github.repository_owner == 'domengabrovsek'
    uses: domengabrovsek/github-actions/.github/workflows/reviewer.yml@main
    with:
      aws_role_arn: ${{ vars.REVIEWER_ROLE_ARN }}
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `aws_role_arn` | required | AWS IAM role ARN to assume via OIDC for reading the Claude token. |

## Onboarding a repo

1. Add `"<repo>": <repo id>` to home-infra's `cloud/reviewer-repos.json` (`gh api repos/domengabrovsek/<repo> --jq .id`). Terraform lets that repo assume the `github-reviewer` role and sets the `REVIEWER_ROLE_ARN` variable, the `safe-to-review` label, and "Allow GitHub Actions to create and approve pull requests". The home-infra GitHub App must be installed on the repo.
2. Add the caller above. Change `branches` when the default branch is not `main`.

## Notes

- Call it with `@main`, including from this repo. The `github-reviewer` role trusts only `reviewer.yml` at `refs/heads/main`, run from a repo listed in `cloud/reviewer-repos.json`, and only reads `/github-reviewer/claude_code_oauth_token`.
- Claude checks each change against agent-config's `AGENTS.md` and `skills/review-pr/checklist.md` from `main`, plus the repo's own `AGENTS.md` or `CLAUDE.md`.
- Fork PRs get a review only when the owner adds the `safe-to-review` label. The run checks out `main`, reads a diff pinned to the labeled commit, removes the label, and skips approval and reply rounds.
- Every job runs on `ubuntu-latest`, because the reviewer holds the token while it reads untrusted PR content.
- Private repos spend Actions minutes on each review.
