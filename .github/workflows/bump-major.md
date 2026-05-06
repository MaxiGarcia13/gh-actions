# Bump major (reusable workflow)

Workflow file: [`.github/workflows/bump-major.yml`](bump-major.yml)

Reusable workflow (`workflow_call`) that runs `npm version major`, creates a tag, and pushes commit + tags.

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

| Input | Default | Description |
| --- | --- | --- |
| `default_branch` | `main` | Branch receiving the bump commit and tags. |
| `node_version` | `20` | Node.js version used by `actions/setup-node`. |
| `working_directory` | `.` | Directory containing `package.json` and `package-lock.json`. |
