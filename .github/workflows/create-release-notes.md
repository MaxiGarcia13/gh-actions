# Create release notes (reusable workflow)

Workflow file: [`.github/workflows/create-release-notes.yml`](create-release-notes.yml)

Reusable workflow (`workflow_call`) that creates a **GitHub Release** for an existing Git tag, using GitHub’s generated release notes (`gh release create --generate-notes`).

## What it does

- Checks out `default_branch` with full history and tags (`fetch-depth: 0`, `fetch-tags: true`).
- Fetches the requested tag from `origin` so it exists locally before creating the release.
- If a release for that tag already exists, the job exits successfully without changing anything.
- Otherwise runs `gh release create <tag> --title "Release <tag>" --generate-notes --verify-tag` against `${{ github.repository }}`.
- Authenticates with **`${{ github.token }}`** (no extra secrets required when nested under another reusable workflow).

The tag must already exist on the remote (for example after `npm version` and `git push --follow-tags`).

## Caller permissions

Workflows that invoke this reusable workflow should grant write access to repository contents so releases can be created:

```yaml
permissions:
  contents: write
```

## Usage in the same repository

```yaml
jobs:
  release-notes:
    uses: ./.github/workflows/create-release-notes.yml
    with:
      tag: v1.2.3
      default_branch: main
```

Typically `tag` comes from another job output (for example after a version bump):

```yaml
jobs:
  bump:
    # ... produce outputs.tag (e.g. v1.2.3)

  release-notes:
    needs: bump
    uses: ./.github/workflows/create-release-notes.yml
    with:
      tag: ${{ needs.bump.outputs.tag }}
      default_branch: main
```

## Usage from another repository

```yaml
jobs:
  release-notes:
    uses: MaxiGarcia13/gh-actions/.github/workflows/create-release-notes.yml@main
    with:
      tag: ${{ needs.bump.outputs.tag }}
      default_branch: main
```

Prefer pinning to a tag or commit SHA instead of `@main` for stable builds.

## Inputs

| Input              | Default | Description                                                                 |
| ------------------ | ------- | --------------------------------------------------------------------------- |
| `tag`              | —       | Git tag for the release (for example `v1.2.3`). Required.                   |
| `default_branch`   | `main`  | Branch to check out first; used to sync refs before fetching the tag.       |

## Integration with bump workflows

[`bump-version.yml`](bump-version.yml) and [`bump-major.yml`](bump-major.yml) call this workflow after bumping and pushing, passing the tag from the [`read-release-tag`](../actions/read-release-tag/README.md) composite action.
