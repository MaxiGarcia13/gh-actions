# Create release notes (reusable workflow)

Workflow file: [`.github/workflows/create-release-notes.yml`](create-release-notes.yml)

Reusable workflow (`workflow_call`) that creates a **GitHub Release** for an existing Git tag. By default it uses GitHub’s generated release notes (`gh release create --generate-notes`). You can also supply a **markdown template file** from the caller repository.

## What it does

- Checks out `default_branch` with full history and tags (`fetch-depth: 0`, `fetch-tags: true`).
- Fetches the requested tag from `origin` and verifies it exists locally.
- If a release for that tag already exists, the job exits successfully without changing anything.
- Otherwise creates a GitHub Release with either:
  - **Generated notes** (`--generate-notes`), or
  - **A template file** (`--notes-file`) with optional `{{generated_notes}}` substitution via the GitHub API.
- Exposes whether a release was created and the release URL as job outputs.
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

## Usage with a release notes template

Add a template file in the **caller repository** (not in this actions repo), then pass its path:

```yaml
jobs:
  release-notes:
    needs: bump
    uses: MaxiGarcia13/gh-actions/.github/workflows/create-release-notes.yml@main
    with:
      tag: ${{ needs.bump.outputs.tag }}
      notes_template: .github/RELEASE_NOTES.template.md
      title: "My App {{tag}}"
```

Example `.github/RELEASE_NOTES.template.md` in the caller repo:

```markdown
# Release {{tag}}

Thanks for using **{{repository}}**.

## What's changed

{{generated_notes}}
```

Supported placeholders:

| Placeholder           | Replaced with                                     |
| --------------------- | ------------------------------------------------- |
| `{{tag}}`             | Release tag (for example `v1.2.3`)                |
| `{{version}}`         | Tag without the leading `v` (for example `1.2.3`) |
| `{{repository}}`      | Repository slug (`owner/name`)                    |
| `{{generated_notes}}` | GitHub-generated changelog for this release       |

When `{{generated_notes}}` appears in the template, the workflow calls the [Generate release notes](https://docs.github.com/en/rest/releases/releases#generate-release-notes-content-for-a-release) API and substitutes the result.

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

| Input             | Default         | Description                                                                                                             |
| ----------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `tag`             | —               | Git tag for the release (for example `v1.2.3`). Required.                                                               |
| `default_branch`  | `main`          | Branch to check out first; used to sync refs before fetching the tag.                                                   |
| `title`           | `Release <tag>` | Release title. Supports `{{tag}}`.                                                                                      |
| `notes_template`  | —               | Path to a markdown template file in the caller repo.                                                                    |
| `notes_start_tag` | —               | Previous tag to start from when generating release notes.                                                               |
| `notes_config`    | —               | Path to a GitHub release notes config file (for example `.github/release.yml`). Used when generating notes via the API. |
| `generate_notes`  | `true`          | Use GitHub-generated notes when no template is provided.                                                                |
| `draft`           | `false`         | Create the release as a draft.                                                                                          |
| `prerelease`      | `false`         | Mark the release as a prerelease.                                                                                       |

## Outputs

| Output        | Description                                                        |
| ------------- | ------------------------------------------------------------------ |
| `created`     | `true` if a new release was created; `false` if it already existed |
| `release_url` | URL of the GitHub Release                                          |

## Integration with bump workflows

[`bump-version.yml`](bump-version.yml) and [`bump-major.yml`](bump-major.yml) call this workflow after bumping and pushing, passing the tag from the [`read-release-tag`](../actions/read-release-tag/README.md) composite action.
