# On PR label usage docs

These docs explain how to use our reusable workflow for acting on specific
GitHub label changes on PRs.

- [On PR label](../.github/workflows/on-pr-label.yml)

This workflow runs when a PR is labeled. It currently supports one use case,
which is to include `(INTERNAL-COMMIT)` in the PR title when the `internal`
label is applied. The purpose of adding `(INTERNAL-COMMIT)` is to exclude
the change from public release notes. When generating release notes, the
system looks for this string to filter out internal commits.

## GitHub workflow permissions

This is the list of the current workflow permissions used as part of this
workflow.

| Permission | Purpose |
| ---------- | ------- |
| `pull-requests: write` | Needed for modifying the PR based on label triggers |

## GitHub secrets

None.

## Usage

In your application repository, create a new file called
`.github/workflows/on-pr-label.yml` (best practice naming).

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

> [!NOTE]
>
> **Label name:** The workflow specifically checks for the `internal` label name
> to trigger the title modification.

## Overrides

None. This workflow does not accept any inputs or overrides.
