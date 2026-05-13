# Deploy to Vercel (reusable workflow)

Workflow file: [`.github/workflows/deploy-to-vercel.yml`](deploy-to-vercel.yml)

Reusable workflow (`workflow_call`) that deploys the project to Vercel production using the Vercel CLI: it runs `vercel pull`, `vercel build --prod`, and `vercel deploy --prebuilt --prod`. Use this when you want GitHub Actions to own production deploys instead of Vercel’s Git integration.

## Disable Vercel’s automatic Git deployments

If the same repository is connected to Vercel, the platform may still create deployments on every push. To turn that off and rely only on this workflow (or other CI), add a `vercel.json` at the repository root with:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "git": {
    "deploymentEnabled": false
  }
}
```

That sets [`git.deploymentEnabled`](https://vercel.com/docs/project-configuration/git-configuration) to `false` so Vercel does not auto-deploy from Git; your Action can still deploy with a token via the CLI.

## What it does

- Checks out the repository.
- Runs `git pull origin main` (expects the default branch to be `main` and appropriate `contents` / network permissions if you rely on this step).
- Sets up Node.js and runs `npm install` in the configured working directory.
- Installs the Vercel CLI globally.
- Runs `vercel pull` for the production environment, then `vercel build --prod`, then `vercel deploy --prebuilt --prod`.

Ensure the Vercel project is linked for CI (for example via `vercel pull` / project settings and environment variables such as org and project IDs where your setup requires them).

## Secrets

| Secret         | Description                                                                                                            |
| -------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `VERCEL_TOKEN` | Vercel [account token](https://vercel.com/account/tokens) used for `vercel pull`, `vercel build`, and `vercel deploy`. |

## Usage in the same repository

```yaml
name: Deploy production

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: ./.github/workflows/deploy-to-vercel.yml
    secrets: inherit
```

## Usage from another repository

```yaml
jobs:
  deploy:
    uses: MaxiGarcia13/gh-actions/.github/workflows/deploy-to-vercel.yml@main
    secrets: inherit
```

Prefer pinning to a tag or commit SHA instead of `@main` for stable builds.

## Inputs

| Input               | Default | Description                                                 |
| ------------------- | ------- | ----------------------------------------------------------- |
| `node_version`      | `24`    | Node.js version used by `actions/setup-node`.               |
| `working_directory` | `.`     | Directory containing `package.json` (useful for monorepos). |

Example with custom inputs:

```yaml
jobs:
  deploy:
    uses: MaxiGarcia13/gh-actions/.github/workflows/deploy-to-vercel.yml@main
    secrets: inherit
    with:
      node_version: "24"
      working_directory: apps/web
```
