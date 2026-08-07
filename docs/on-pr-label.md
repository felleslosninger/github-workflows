# On PR label usage docs

These docs explain how to use our reusable workflow for acting on specific
GitHub label changes on PRs

- [On PR label](../.github/workflows/on-pr-label.yml)

This workflow runs when a PR is labeled. It currently support one use-case,
which is to include `(INTERNAL-COMMIT)` in the PR title of a PR that is updated
with the `internal` label. The purpose of adding `(INTERNAL-COMMIT)` in a PR
title is to exclude this release from the public release notes of a specific
app. When publishing release notes we look for the `(INTERNAL-COMMIT)` string in
a commit to decide whether or not to exclude it. This is to avoid publishing
release notes for commits that we do not want the public to see.

## Usage

In your application repository, create a new file called
`.github/workflows/on-pr-label.yml` (best practice naming)

```yaml
name: On PR label

on:
  pull_request:
    types: [labeled]

jobs:
  on-pr-label:
    uses: felleslosninger/github-workflows/.github/workflows/on-pr-label.yml@main
    permissions:
      pull-requests: write
```

## Overrides

None
