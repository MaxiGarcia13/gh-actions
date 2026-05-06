# gh-actions

Reusable [GitHub Actions workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) and actions that can be consumed from this repository or external repositories via `jobs.<job_id>.uses`.

## Available automations

- Reusable workflow: [`bump-version`](.github/workflows/bump-version.md)
  - File: `.github/workflows/bump-version.yml`
  - Purpose: bumps `package.json` version, creates a release tag, and pushes commit + tags.
- Reusable workflow: [`bump-major`](.github/workflows/bump-major.md)
  - File: `.github/workflows/bump-major.yml`
  - Purpose: bumps `package.json` major version, creates a release tag, and pushes commit + tags.
- Composite action: [`configure-git-author`](.github/actions/configure-git-author/README.md)
  - File: `.github/actions/configure-git-author/action.yml`
  - Purpose: sets `git config user.name` and `git config user.email` in workflow jobs.
