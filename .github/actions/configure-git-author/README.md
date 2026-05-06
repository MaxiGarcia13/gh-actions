# Configure git author (composite action)

Configures git commit identity for workflows that create commits or tags.

## What it does

- Sets `git config user.name` from `user_name` input
- Sets `git config user.email` from `user_email` input

## Usage

```yaml
- name: Configure git author
  uses: ./.github/actions/configure-git-author
  with:
    user_name: github-actions[bot]
    user_email: github-actions[bot]@users.noreply.github.com
```

## Use from another repository

```yaml
jobs:
  configure-git-author:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure git author
        uses: MaxiGarcia13/gh-actions/.github/actions/configure-git-author@main
        with:
          user_name: github-actions[bot]
          user_email: github-actions[bot]@users.noreply.github.com
```

## Inputs

| Input        | Required | Default                                        | Description                        |
| ------------ | -------- | ---------------------------------------------- | ---------------------------------- |
| `user_name`  | No       | `github-actions[bot]`                          | Value for `git config user.name`.  |
| `user_email` | No       | `github-actions[bot]@users.noreply.github.com` | Value for `git config user.email`. |

## Notes

- This action is intended for use after checkout in jobs that run git commit/tag commands.
