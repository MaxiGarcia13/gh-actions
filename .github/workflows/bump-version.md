# Bump version (reusable workflow)

Workflow file: [`.github/workflows/bump-version.yml`](bump-version.yml)

Reusable workflow (`workflow_call`) that bumps the caller repository version with `npm version`, creates a tag, and pushes commit + tags to the target branch.

It runs `npm ci`, so `package-lock.json` must exist in the selected working directory.

## What it does

- Checks out with full history (`fetch-depth: 0`) and fetches tags.
- Reads commit subjects between the latest `v*` tag (if present) and `HEAD`.
- Chooses bump level:
  - `minor` by default
  - `patch` if any commit subject matches conventional-commit `fix` format (`fix:` / `fix(scope):`, optional `!`)
- Runs `npm version <minor|patch> -m "chore(release): v%s"`.
- Pushes `HEAD` and tags to `default_branch`.
- When `use_release_notes` is `true` (default), calls [`create-release-notes.yml`](create-release-notes.yml) to create a GitHub Release for the new tag.

## Skip behavior

The job is skipped when:

- the event is `push`, and
- the head commit message starts with `chore(release):`

This prevents infinite loops after the workflow creates the release commit.

## Usage in the same repository

Create a caller workflow (for example, on push to `main`) and reference this reusable workflow by relative path:

```yaml
name: Bump version on main

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  bump:
    uses: ./.github/workflows/bump-version.yml
    secrets: inherit
```

## Usage from another repository

```yaml
name: Bump version on main

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  bump:
    uses: MaxiGarcia13/gh-actions/.github/workflows/bump-version.yml@main
    secrets: inherit
```

Prefer pinning to a tag or commit SHA instead of `@main` for stable builds.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `default_branch` | `main` | Branch receiving the bump commit and tags (`git push origin HEAD:<branch> --follow-tags`). |
| `node_version` | `24` | Node.js version used by `actions/setup-node`. |
| `working_directory` | `.` | Directory containing `package.json` and `package-lock.json` (useful for monorepos). |
| `use_release_notes` | `true` | When `true`, runs the `release-notes` job after the bump to create a GitHub Release via [`create-release-notes.yml`](create-release-notes.yml). Set to `false` to skip release creation. |
| `lfs` | `false` | When `true`, fetches Git LFS files during checkout (`actions/checkout` `lfs` input). |

Example with custom inputs:

```yaml
jobs:
  bump:
    uses: MaxiGarcia13/gh-actions/.github/workflows/bump-version.yml@main
    secrets: inherit
    with:
      default_branch: main
      node_version: "24"
      working_directory: packages/app
```

To bump without creating a GitHub Release:

```yaml
jobs:
  bump:
    uses: MaxiGarcia13/gh-actions/.github/workflows/bump-version.yml@main
    secrets: inherit
    with:
      use_release_notes: false
```

Release note customization (templates, title, draft/prerelease) is configured on [`create-release-notes.yml`](create-release-notes.yml). These bump workflows only toggle whether that step runs.

## Outputs

| Output | Description |
| --- | --- |
| `tag` | Release tag created by the bump (for example `v1.2.3`). Empty when the bump job is skipped. |

Downstream jobs in the caller workflow can read it as `needs.bump.outputs.tag`:

```yaml
jobs:
  bump:
    uses: MaxiGarcia13/gh-actions/.github/workflows/bump-version.yml@main
    secrets: inherit

  deploy:
    needs: bump
    runs-on: ubuntu-latest
    steps:
      - run: echo "Released ${{ needs.bump.outputs.tag }}"
```

## Concurrency

Uses the group `bump-version-${{ github.repository }}-${{ github.ref }}` with `cancel-in-progress: false`, so concurrent runs queue instead of cancelling each other.
