# Bump major (reusable workflow)

Workflow file: [`.github/workflows/bump-major.yml`](bump-major.yml)

Reusable workflow (`workflow_call`) that runs `npm version major`, creates a tag, and pushes commit + tags.

When `use_release_notes` is `true` (default), it also calls [`create-release-notes.yml`](create-release-notes.yml) to create a GitHub Release for the new tag.

## Usage in the same repository

```yaml
name: Bump major on main

on:
  workflow_dispatch:

permissions:
  contents: write

jobs:
  bump:
    uses: ./.github/workflows/bump-major.yml
    secrets: inherit
```

## Usage from another repository

```yaml
name: Bump major on main

on:
  workflow_dispatch:

permissions:
  contents: write

jobs:
  bump:
    uses: MaxiGarcia13/gh-actions/.github/workflows/bump-major.yml@main
    secrets: inherit
```

Prefer pinning to a tag or commit SHA instead of `@main` for stable builds.

## Inputs

| Input               | Default | Description                                                  |
| ------------------- | ------- | ------------------------------------------------------------ |
| `default_branch`    | `main`  | Branch receiving the bump commit and tags.                   |
| `node_version`      | `24`    | Node.js version used by `actions/setup-node`.                |
| `working_directory` | `.`     | Directory containing `package.json` and `package-lock.json`. |

To bump without creating a GitHub Release:

```yaml
jobs:
  bump:
    uses: MaxiGarcia13/gh-actions/.github/workflows/bump-major.yml@main
    secrets: inherit
    with:
      use_release_notes: false
```

Release note customization (templates, title, draft/prerelease) is configured on [`create-release-notes.yml`](create-release-notes.yml). These bump workflows only toggle whether that step runs.

## Outputs

| Output | Description                                             |
| ------ | ------------------------------------------------------- |
| `tag`  | Release tag created by the bump (for example `v2.0.0`). |

Downstream jobs in the caller workflow can read it as `needs.bump.outputs.tag`.
