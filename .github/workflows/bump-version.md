# Bump version (reusable workflow)

Workflow: [`.github/workflows/bump-version/action.yml`](action.yml)

Reusable workflow (`workflow_call` only): it bumps the **caller** repo’s `package.json` with `npm version`, creates a git tag, and pushes to the branch you configure. It uses `npm ci`, so a committed **`package-lock.json`** next to `package.json` is required.

## When the job runs

The bump job is skipped if:

- `github.actor` is `github-actions[bot]`, or  
- the event is **`push`** and the head commit message starts with `chore(release):`

For reuse, the [`github` context is the caller’s run](https://docs.github.com/en/actions/reference/workflows-and-actions/reusing-workflow-configurations#github-context), so you do not need an extra `if` on the calling job.

## Use in this repository (push to `main`)

Add a small caller workflow under `.github/workflows/` (for example `bump-version-on-push.yml`) that triggers on `push` to `main` and calls this file with a **relative** path:

```yaml
name: Bump version on main

on:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  bump:
    uses: ./.github/workflows/bump-version/action.yml
    secrets: inherit
```

## Use from another repository

```yaml
name: Bump version on main

on:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  bump:
    uses: MaxiGarcia13/gh-actions/.github/workflows/bump-version/action.yml@main
    secrets: inherit
```

Pin `@main` to a tag or commit SHA when you care about stability. Source repo: [MaxiGarcia13/gh-actions](https://github.com/MaxiGarcia13/gh-actions).

### Inputs

| Input | Default | Description |
| --- | --- | --- |
| `default_branch` | `main` | Branch for `git push origin HEAD:<branch>`. |
| `node_version` | `20` | Node version for `actions/setup-node`. |
| `working_directory` | `.` | Directory that contains `package.json` (monorepos). |

```yaml
jobs:
  bump:
    uses: MaxiGarcia13/gh-actions/.github/workflows/bump-version/action.yml@main
    secrets: inherit
    with:
      default_branch: main
      node_version: "22"
      working_directory: packages/app
```

## Bump rules

Between the latest `v*` tag (if any) and `HEAD`, commit subjects are scanned:

- Default: **minor** (`npm version minor`).
- **Patch** if any subject matches conventional-commit **fix** (`fix:` / `fix(scope):`, optional `!`).

Version commit message: `chore(release): v%s` (npm substitutes the new version).

## Concurrency

Group: `bump-version-${{ github.repository }}-${{ github.ref }}`, `cancel-in-progress: false` (runs queue instead of cancelling each other).

## About this path

This workflow and README live under `.github/workflows/bump-version/`. Callers reference **`.github/workflows/bump-version/action.yml@REF`**. GitHub’s docs recommend keeping workflow YAML directly under `.github/workflows/`; if you hit path or reuse issues, move the file to the workflows root and update `uses:` paths accordingly.
