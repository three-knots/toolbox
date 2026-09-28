# GitHub repo bootstrap

Steps and files for setting up a GitHub repo that was created in the UI but has no commits yet. Copy the commands, replace the placeholders, and run them in order.

Captured from `Skylight-Health/dtc_ads` on 2026-09-28. Where this kit differs from that repo, the difference is noted.

## What's in here

| File | What it is |
| --- | --- |
| [repo-settings.json](repo-settings.json) | General repo settings: features, merge buttons, auto-delete branches. |
| [actions-workflow-permissions.json](actions-workflow-permissions.json) | Default `GITHUB_TOKEN` permissions for Actions (read-only, can't approve PRs). |
| [protect-main.json](protect-main.json) | Ruleset that protects the default branch. |
| [CODEOWNERS](CODEOWNERS) | Starter `.github/CODEOWNERS`. Required by the ruleset. |
| [terraform/](terraform/) | Optional. GitHub Actions workflows that plan and apply Terraform to dev/prod through AWS OIDC. |

## Prerequisites

- `gh` CLI, logged in as a repo admin (`gh auth status`).
- The repo already exists on GitHub and is empty.

Set these once in your shell. Every command below uses them.

```bash
REPO=<owner>/<repo>          # e.g. Skylight-Health/dtc_ads
TOOLBOX=<path-to-toolbox>/github
```

## Steps

### 1. Apply repo settings

```bash
gh api -X PATCH "repos/$REPO" --input "$TOOLBOX/repo-settings.json"
gh api -X PUT "repos/$REPO/actions/permissions/workflow" --input "$TOOLBOX/actions-workflow-permissions.json"
```

**Expected:** each command prints the updated JSON. Spot-check it with:

```bash
gh api "repos/$REPO" --jq '{delete_branch_on_merge, allow_auto_merge, allow_squash_merge}'
```

Visibility (public/private) is set when the repo is created, not here.

### 2. Make the first commit

Protect the branch only after it exists. Otherwise the pull-request rule can block the initial push.

```bash
git clone "git@github.com:$REPO.git" && cd "$(basename "$REPO")"
mkdir -p .github
cp "$TOOLBOX/CODEOWNERS" .github/CODEOWNERS   # then edit the owners
echo "# $(basename "$REPO")" > README.md
git add . && git commit -m "Initial commit" && git push -u origin main
```

If this is a Terraform repo, do the [terraform/](terraform/) setup here, before the first push.

### 3. Protect `main`

```bash
gh api -X POST "repos/$REPO/rulesets" --input "$TOOLBOX/protect-main.json"
```

**Expected:** JSON with an `id` and `"enforcement": "active"`. Confirm that it applies to `main`:

```bash
gh api "repos/$REPO/rules/branches/main" --jq '.[].type'
# deletion, non_fast_forward, pull_request
```

The ruleset:

- Blocks deleting `main` and force-pushing to it.
- Requires a PR with 1 approval to merge into `main`.
- Dismisses approvals when new commits are pushed.
- Requires a review from a code owner (so `.github/CODEOWNERS` must exist).
- Lets repository admins bypass all of the above.

> **Differs from dtc_ads.** dtc_ads uses classic branch protection instead of a ruleset. Its settings also differ in two ways: `require_code_owner_review` is `false` and `require_last_push_approval` is `true`. With last-push approval on, someone other than the last person who pushed has to approve. Edit `protect-main.json` to match either way.

### 4. Add Actions secrets

Secrets aren't copied between repos. Set each one by hand, and never commit the values.

```bash
gh secret set <NAME> --repo "$REPO"      # prompts for the value
```

dtc_ads has `AWS_ROLE_ARN_DEV` and `AWS_ROLE_ARN_PROD`. They're only needed for the [terraform/](terraform/) workflows.

### 5. Access

Add people or teams in **Settings → Collaborators and teams**, or from the CLI:

```bash
gh api -X PUT "repos/$REPO/collaborators/<username>" -f permission=admin     # pull|triage|push|maintain|admin
gh api -X PUT "orgs/<org>/teams/<team-slug>/repos/$REPO" -f permission=push
```

### 6. Security features (optional)

All of these are **off** in dtc_ads. To turn on Dependabot alerts and automatic security fixes:

```bash
gh api -X PUT "repos/$REPO/vulnerability-alerts"
gh api -X PUT "repos/$REPO/automated-security-fixes"
```

Secret scanning and push protection on private repos need GitHub Advanced Security. Turn them on in **Settings → Code security** if the org has it.

## Left at GitHub defaults

These were checked in dtc_ads and need no setup:

- **Labels:** the default nine.
- **Everything else:** no environments, webhooks, autolinks, topics, custom properties or Actions variables.
- **Actions:** enabled, all actions allowed.
