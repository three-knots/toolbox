# toolbox

Templates, patterns, workflows and processes I reach for again and again. Everything here is meant to be read by a person and copied into another repo, wiki or terminal. Nothing in this repo runs on its own.

## What's in here

| Folder | What it's for |
| --- | --- |
| [github/](github/) | Setting up a new GitHub repo: repo settings, Actions permissions, a `main` branch ruleset and a CODEOWNERS file, applied with `gh`. |
| [github/terraform/](github/terraform/) | GitHub Actions workflows that plan and apply Terraform to `dev` and `prod` AWS accounts through OIDC. |
| [runbooks/](runbooks/) | A starter kit for on-call runbooks: a landing page, a blank template and a worked example. |

## How to use it

1. **Start at the folder's README.** Every folder has one. It says what the kit is for, lists each file and where it goes, and gives the steps in order.
2. **Copy, don't link.** Copy files or snippets into your project and adapt them there. The toolbox is a reference, not a dependency, so changes here won't reach copies you've already made.
3. **Fill in the blanks.** Look for these before you commit:
   - `<placeholders>` in angle brackets, like `<owner>/<repo>` or `@<org>/<team>`.
   - `# EDIT` comments on values that differ per project, like AWS region or tool versions.
   - `>` quoted guidance notes in document templates. Delete them once the section is filled in.
4. **Read the caveats.** READMEs call out known trade-offs ("Things to know before adopting this") and where a template differs from the repo it was taken from. Read them before you adopt a kit.

### Copying with the command line

READMEs that include shell steps assume a `TOOLBOX` variable pointing at the relevant folder of your local clone:

```bash
git clone git@github.com:<owner>/toolbox.git ~/toolbox
TOOLBOX=~/toolbox/github
cp "$TOOLBOX/CODEOWNERS" .github/CODEOWNERS
```

### Files stored under a different name

Some files can't be stored under their real name here without taking effect in this repo. They're renamed, and the README's file table shows where each one goes:

| Stored here as | Copy to |
| --- | --- |
| `gitignore` | `.gitignore` |
| `workflows/*.yml` | `.github/workflows/*.yml` |
| `CODEOWNERS` (inside `github/`) | `.github/CODEOWNERS` |

This repo's own `.gitignore` ignores `.github/`, so nothing in it ever runs as a workflow here.

## Adding something new

1. **Pick a folder by topic.** Create one if nothing fits. Nest a subfolder when a kit is an optional add-on to another, the way `github/terraform/` extends `github/`.
2. **Write the README first.** Follow the shape of the existing ones:
   - one or two sentences on what it is and when to use it
   - where it came from, if it was taken from a real project, and any deliberate differences
   - a table of files: what each one is and where it goes
   - prerequisites, then numbered steps with the exact commands and the expected result of each
   - known trade-offs or gotchas
3. **Make the templates copy-ready.** Use `<placeholders>` and `# EDIT` markers, strip anything project-specific or secret, and rename dotfiles as described above.
4. **Link it from the table** in [What's in here](#whats-in-here).

## License

[GPL-3.0](LICENSE)
