---
name: commits
description: Use Conventional Commits format for all git commits. Apply when creating commits or asking about commit messages.
---

# Conventional Commits

## Format

```
<type>: <description>
```

## Types

- **feat**: New feature
- **fix**: Bug fix
- **refactor**: Code refactoring (no feature or fix)
- **docs**: Documentation changes
- **style**: Code style changes (formatting, semicolons, etc.)
- **test**: Adding or updating tests
- **chore**: Build process, dependencies, tools
- **perf**: Performance improvements
- **ci**: CI/CD configuration changes

## Rules

- Use lowercase for type and description
- Keep description under 72 characters
- Use imperative mood ("add" not "added" or "adds")
- No period at the end of description

## Examples

```
feat: add user booking calendar
fix: resolve timezone issue in scheduling
refactor: simplify booking validation logic
docs: update API documentation
chore: update dependencies to latest versions
```

## When to use

- Creating git commits
- Asking about commit message format
- Writing commit messages in the project
