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

## GitHub Environments and Deployments

The deploy job uses a [GitHub Environment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) so runs appear under the repository’s **Environments** / deployment activity. Configure or review the environment under **Settings → Environments** (for example create `production` before first use).

- **`deployment_environment`** — Name of that GitHub Environment (default `production`).
- **`deployment_url`** — Optional stable URL shown as the environment link in the GitHub UI (for example your production domain or Vercel alias). Leave empty to record the deployment without a URL. The CLI does not feed this value automatically; pass whatever URL you want callers to open.

You cannot set `environment` on the **caller** job that invokes `workflow_call`; GitHub only allows a limited set of keys there. Pass these inputs via `with:` so the reusable workflow attaches the environment to the actual deploy job.

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
    with:
      deployment_environment: production
      deployment_url: https://your-production-domain.example
```

## Usage from another repository

```yaml
jobs:
  deploy:
    uses: MaxiGarcia13/gh-actions/.github/workflows/deploy-to-vercel.yml@main
    secrets: inherit
    with:
      deployment_url: https://your-production-domain.example
```

Prefer pinning to a tag or commit SHA instead of `@main` for stable builds. Omit `with` entirely to use defaults (`deployment_environment: production`, no URL).

## Inputs

| Input                     | Default        | Description                                                                                      |
| ------------------------- | -------------- | ------------------------------------------------------------------------------------------------ |
| `node_version`            | `24`           | Node.js version used by `actions/setup-node`.                                                    |
| `working_directory`       | `.`            | Directory containing `package.json` (useful for monorepos).                                      |
| `deployment_environment`  | `production`   | GitHub Environment name for this job (Deployments / Environments UI).                            |
| `deployment_url`          | _(empty)_      | Optional URL for the environment link in GitHub; omit or leave empty to skip the link.           |

Example with custom inputs:

```yaml
jobs:
  deploy:
    uses: MaxiGarcia13/gh-actions/.github/workflows/deploy-to-vercel.yml@main
    secrets: inherit
    with:
      node_version: "24"
      working_directory: apps/web
      deployment_environment: production
      deployment_url: https://your-app.vercel.app
```
