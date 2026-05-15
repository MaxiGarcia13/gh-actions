# Read release tag (composite action)

Reads `version` from `package.json` and exposes it as an output with a `v` prefix (for example `v1.2.3`), matching typical semver Git tags.

## What it does

- Runs in `working_directory` (relative to the workspace root).
- Uses Node.js (`node -p "require('./package.json').version"`) to read the version field.
- Sets output `tag` to `v` concatenated with that version.

## Requirements

- **Node.js** must be available on the runner (for example via `actions/setup-node`) before this step runs.

## Usage

```yaml
- name: Setup Node
  uses: actions/setup-node@v4
  with:
    node-version: "24"

- name: Read release tag
  id: release_tag
  uses: ./.github/actions/read-release-tag
  with:
    working_directory: .

- name: Example
  run: echo "${{ steps.release_tag.outputs.tag }}"
```

### Monorepo package directory

```yaml
- name: Read release tag
  id: release_tag
  uses: ./.github/actions/read-release-tag
  with:
    working_directory: packages/my-package
```

## Use from another repository

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "24"

      - name: Read release tag
        id: release_tag
        uses: MaxiGarcia13/gh-actions/.github/actions/read-release-tag@main
        with:
          working_directory: .

      - name: Example
        run: echo "${{ steps.release_tag.outputs.tag }}"
```

Prefer pinning to a tag or commit SHA instead of `@main` for stable builds.

## Inputs

| Input                | Required | Default | Description                                      |
| -------------------- | -------- | ------- | ------------------------------------------------ |
| `working_directory`  | No       | `.`     | Directory that contains `package.json`.          |

## Outputs

| Output | Description                                      |
| ------ | ------------------------------------------------ |
| `tag`  | Release tag string (`v` + `package.json` version). |
