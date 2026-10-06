# CLAUDE.md

This repo is a **reference library**: templates, patterns, workflows and processes that an engineer reads and copies into another project. Nothing here runs, deploys or gets imported. The reader is a person copying by hand, so every change is judged by one test: can an engineer who has never seen this repo copy it and use it without asking anyone?

Read [README.md](README.md) before adding or restructuring anything. It is the single source of truth for the folder layout, the copy conventions (`<placeholders>`, `# EDIT` markers, renamed dotfiles, the `TOOLBOX` variable) and the checklist for adding a new kit.

## Working in this repo

- **Keep each README in step with its files.** When you add, rename or change a template, update the file table, the steps and the root README's "What's in here" table in the same change. A README that describes a file differently from its contents is a bug.
- **Check every claim against the files.** Before a README says a file contains or does something, open the file and confirm it. Commands in READMEs are exact and copy-pasteable, and each one is followed by its expected result.
- **Keep templates copy-ready.** Replace project-specific values, account IDs, ARNs and secrets with `<placeholders>`, and mark values that differ per project with `# EDIT`. Keep provenance notes ("Captured from …", "Differs from …") and the "Things to know before adopting this" caveats. They are how a reader decides whether a kit fits their project.
- **Store files that would activate under their real name under a neutral name.** For example, store `gitignore` rather than `.gitignore`, and keep workflows in `workflows/` rather than `.github/workflows/`. Say where each one goes in the README's file table. This repo's `.gitignore` ignores `.github/`, so anything placed there is silently dropped.
- **Write for a human reader.** Use plain language and short sentences, put the most important steps first, and link to the specific section of external docs.
