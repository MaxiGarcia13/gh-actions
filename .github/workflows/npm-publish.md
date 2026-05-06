# Publish to npm (reusable workflow)

Workflow file: [`.github/workflows/npm-publish.yml`](npm-publish.yml)

Reusable workflow (`workflow_call`) that installs dependencies and publishes the package to npm by running an npm script command.

## What it does

- Checks out `main` with full history (`fetch-depth: 0`).
- Sets up Node.js `24` and configures npm registry auth for `registry.npmjs.org`.
- Runs `npm ci`.
- Runs `npm run <command>` where `command` is the provided input.
- Uses `secrets.NPM_TOKEN` as `NODE_AUTH_TOKEN` for npm authentication.

## Required secret

| Secret      | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `NPM_TOKEN` | npm automation token with permission to publish the package. |

## Usage in the same repository

```yaml
name: Release

on:
  workflow_dispatch:

jobs:
  bump:
    uses: ./.github/workflows/bump-version.yml
    secrets: inherit

  publish:
    needs: bump
    uses: ./.github/workflows/npm-publish.yml
    secrets: inherit
    with:
      command: publish:npm
```

## Usage from another repository

```yaml
name: Release

on:
  workflow_dispatch:

jobs:
  publish:
    needs: bump
    uses: MaxiGarcia13/gh-actions/.github/workflows/npm-publish.yml@main
    secrets: inherit
    with:
      command: publish:npm
```

Prefer pinning to a tag or commit SHA instead of `@main` for stable builds.

## Inputs

| Input     | Default       | Description                                      |
| --------- | ------------- | ------------------------------------------------ |
| `command` | `publish:npm` | npm script name executed as `npm run <command>`. |
