# pr-author-limit

GitHub Actions workflow that auto-closes new pull requests when the author already has more than N open PRs in the same repository.

Particularly useful for tackling **AI-generated PRs** that can flood a repository with concurrent submissions faster than a human can review them.

## What it does

When a PR is opened or reopened:

1. Counts how many PRs the same author currently has open.
2. If the count exceeds the configured limit, the workflow:
   - Posts a comment on the new PR explaining why it's being closed.
   - Closes the PR.

Existing open PRs from that author are left alone: only the new one is closed.

## Install

Copy `.github/workflows/pr-author-limit.yml` into the `.github/workflows/` directory of your repository and commit it to your default branch.

> The workflow uses `pull_request_target`, so it must exist on the default branch to take effect. Changes inside a PR do not run this workflow until merged.

## Configure the limit

The limit is read from a **repository variable** named `MAX_OPEN_PRS_PER_AUTHOR`. Default is `5` if the variable is not set.

To change it:

1. Go to **Settings → Secrets and variables → Actions → Variables**.
2. Click **New repository variable**.
3. Name: `MAX_OPEN_PRS_PER_AUTHOR`
4. Value: any non-negative integer

No workflow restart is needed. The next PR opened will pick up the new value.

## Permissions

The workflow requests `pull-requests: write` on the built-in `GITHUB_TOKEN`. No PAT or secret is required.

`pull_request_target` runs in the context of the base branch, so the token has write access even for PRs from forks. The workflow does not check out PR code, so there is no risk of executing untrusted code from a fork.

## Behavior notes

- Triggers: `opened` and `reopened` PRs only. Existing PRs are not affected retroactively.
- The author count includes the newly opened PR itself.
- All open PRs are fetched via `github.paginate` (100 per page, no hard cap). Note GitHub's REST API limits `per_page` to 100, so pagination is mandatory above that.
- Bot accounts (e.g. `dependabot[bot]`, `github-actions[bot]`) are subject to the same limit. Add an early-return filter on `author` if you want to exempt specific accounts.

## License

See [LICENSE](LICENSE).
