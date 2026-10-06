# Terraform CI workflows

GitHub Actions workflows that plan and apply Terraform to a `dev` and a `prod` AWS account. They authenticate to AWS through OIDC, so no long-lived keys are stored in GitHub. Taken from `Skylight-Health/dtc_ads`.

## What happens when

| Event | dev | prod |
| --- | --- | --- |
| PR opened or updated against `main` | plan, then **apply** | plan |
| Merge to `main` | plan, then apply | plan, then apply (after dev succeeds) |

Each PR gets one plan summary comment per environment. It's updated in place on every push, not reposted.

**Things to know before adopting this:**
- **Dev is applied from unmerged PR branches.** Two open PRs can overwrite each other's dev changes. To make dev change only on merge, delete the `apply-dev` job from `terraform-pr.yml`.
- **Prod applies on merge with no manual approval step.** To add one, create a GitHub Environment named `prod` with required reviewers and add `environment: ${{ inputs.environment }}` to the apply job. Required reviewers on private repos need a GitHub Team or Enterprise plan.
- **The apply job re-plans instead of applying the plan that was reviewed.** If anything changes between the plan and the apply, the apply includes it.

## Prerequisites

- The repo is set up with the [GitHub repo bootstrap](../README.md), or you have `REPO` and `TOOLBOX` set as described in its [Prerequisites](../README.md#prerequisites). The commands below use both.
- The [AWS setup](#aws-setup-per-environment) below already exists for each environment.

## Files

Copy these into the new repo:

| From here | To the new repo |
| --- | --- |
| [workflows/_terraform-plan.yml](workflows/_terraform-plan.yml) | `.github/workflows/_terraform-plan.yml` |
| [workflows/_terraform-apply.yml](workflows/_terraform-apply.yml) | `.github/workflows/_terraform-apply.yml` |
| [workflows/terraform-pr.yml](workflows/terraform-pr.yml) | `.github/workflows/terraform-pr.yml` |
| [workflows/terraform-main.yml](workflows/terraform-main.yml) | `.github/workflows/terraform-main.yml` |
| [gitignore](gitignore) | `.gitignore` |

```bash
mkdir -p .github/workflows
cp "$TOOLBOX/terraform/workflows/"*.yml .github/workflows/
cp "$TOOLBOX/terraform/gitignore" .gitignore
```

Then edit the `# EDIT` lines at the top of the two `_terraform-*.yml` files: `AWS_REGION` and `TF_VERSION`.

## Repo layout the workflows expect

```
environments/
  dev/     provider.tf (S3 backend), main.tf, ...
  prod/
modules/   (optional) shared modules
```

Terraform runs in `environments/<env>/`. Each environment has its own S3 backend. In dtc_ads the backend looks like this:

```hcl
backend "s3" {
  bucket       = "terraform.<env>.skylight.health"
  key          = "<repo>/terraform.tfstate"
  region       = "us-east-2"
  encrypt      = true
  use_lockfile = true   # S3-native locking; no DynamoDB table needed
}
```

## AWS setup (per environment)

The workflows expect this AWS setup to already exist:

1. **State bucket** in the target account, with versioning and default encryption on.
2. **GitHub OIDC provider and CI role.** For dtc_ads these are managed in `terraform_iac`, not in the app repo. The role's trust policy must allow the subject `repo:<owner>/<repo>:*`. Watch the exact repo spelling; `dtc_ads` uses an underscore.
3. **Repo secret** holding the role ARN:

   ```bash
   gh secret set AWS_ROLE_ARN_DEV  --repo "$REPO" --body "arn:aws:iam::<dev-account-id>:role/<role-name>"
   gh secret set AWS_ROLE_ARN_PROD --repo "$REPO" --body "arn:aws:iam::<prod-account-id>:role/<role-name>"
   ```

## Changes from the dtc_ads original

- Region and Terraform version are set once in a top-level `env:` block instead of being repeated in each step.
- **Bug fix:** the original plan step ran `terraform plan | tee` without `pipefail`. The pipeline's result came from `tee`, which always succeeds, so a failed plan still showed ✅ and the job passed. The templates set `shell: bash` so the job fails when the plan fails. dtc_ads still has this bug.
- `plan_output.txt` is added to `.gitignore`.
