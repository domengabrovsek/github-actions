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

# contents: write is for the shared workflow's reply jobs, which resolve and
# reopen review threads and run no PR code. The Claude review job keeps
# contents: read.
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

## Jobs

| Job | Runs on | Does | Permissions |
| --- | --- | --- | --- |
| `Review` | `pull_request`, or `pull_request_target` with the `safe-to-review` label | Reviews the whole diff and posts inline comments | `contents: read`, `pull-requests: write`, `id-token: write` |
| `Post review replies` | After `Review` | Posts Claude's replies to earlier threads and reopens them | `contents: write`, `pull-requests: write` |
| `Reply` | `pull_request_review_comment` from the owner on a same-repo PR | Drafts a reply in a thread Claude started | `contents: read`, `pull-requests: read`, `id-token: write` |
| `Post reply` | After `Reply` | Posts the reply and resolves the thread when Claude judges it settled | `contents: write`, `pull-requests: write` |
| `Approve` | After a review or reply round | Approves when no Claude thread is open | `contents: read`, `pull-requests: write` |

Each job sets its own permissions, and the caller's grant is the ceiling. The caller grants `contents: write` only because GitHub requires it to resolve or reopen a review thread. The two reply-posting jobs are the only ones that get it. They run no PR code and never see the Claude token. The jobs that hold the token keep `contents: read`.

## Onboarding a repo

1. Add `"<repo>": <repo id>` to [`cloud/reviewer-repos.json`](https://github.com/domengabrovsek/home-infra/blob/main/cloud/reviewer-repos.json) in the home-infra repo (`gh api repos/domengabrovsek/<repo> --jq .id`). Terraform lets that repo assume the `github-reviewer` role and sets the `REVIEWER_ROLE_ARN` variable, the `safe-to-review` label, and "Allow GitHub Actions to create and approve pull requests". The home-infra GitHub App must be installed on the repo.
2. Once that apply finishes, add the caller above in its own PR. It fails until the apply has set the variable and the trust. Change `branches` when the default branch is not `main`.

## Notes

- Call it with `@main`, including from this repo. The `github-reviewer` role trusts only `reviewer.yml` at `refs/heads/main`, run from a repo listed in `cloud/reviewer-repos.json`, and only reads `/github-reviewer/claude_code_oauth_token`.
- Claude checks each change against agent-config's `AGENTS.md` and `skills/review-pr/checklist.md` from `main`, plus the repo's own `AGENTS.md` or `CLAUDE.md`.
- A `pull_request` run from a fork gets no OIDC token, so it cannot read the Claude token. Fork PRs get a review only when the owner adds the `safe-to-review` label, which starts a `pull_request_target` run. The `Review` job runs only when the label sender is the repo owner. The run checks out `main`, reads a diff pinned to the labeled commit, removes the label, and skips reply rounds. A clean review approves, as on same-repo PRs.
- Claude's approval never merges anything by itself while the owner is the only writer. Before giving another account write access, require a code-owner review so a steered approval cannot satisfy branch protection.
- No job executes PR code. The review reads the diff and files as data. Every job runs on GitHub-hosted `ubuntu-latest`, because the reviewer holds the token while it reads untrusted PR content, and a hosted runner is discarded after the job.
- Claude runs with `--model opus` and gets no shell, network, or file-write tools. It reads the checkout and PR data that earlier steps fetched to files.
- Draft PRs get no review until they are ready.
- Claude answers only the owner's replies, only in threads Claude started, and stops after five replies in one thread.
- A fork review fails when the PR head moved past the labeled commit, because Claude's comments land on the live head. Add the label again to review the new push.
- The approval is pinned to the reviewed commit. Each push starts a new review of the whole diff, and branch protection dismisses the stale approval, so an approval always covers the latest commit. When a review opens or reopens a thread, the `Approve` job dismisses Claude's standing approvals and approves again once no thread is open (`reviewer.yml:9-11`, `reviewer.yml:23-25`).
- Private repos spend Actions minutes on each review.
