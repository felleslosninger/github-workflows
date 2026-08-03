# PR checks usage docs (image)

These docs explain how to use our reusable workflow for PRs in app repos that
produces container images

- [PR checks for container images](../.github/workflows/ci-pr-checks-image.yml)

This workflow support

- PR title validation
- Trivy image scanning
- Automatic merge of Dependabot PRs

## Usage

In your application repository, create a new file called
`.github/workflows/pull-request.yml` (best practice naming) and override the
application type

```yaml
name: Pull request

on:
  pull_request:
    types:
      - opened
      - reopened
      - synchronize
      - edited
    branches:
      - main

jobs:
  pull-request-checks:
    uses: felleslosninger/github-workflows/.github/workflows/ci-pr-checks-image.yml@main
    permissions:
      contents: write
      pull-requests: write
    with:
      application-type: spring-boot | quarkus | docker
    secrets: inherit
```

> [!NOTE]
> The workflow trigger types are needed to also trigger on `edited`, which
> ensures that the workflow runs when the PR title is updated (in the case of PR
> title validation failures).

You do not need to override anything else to start using this workflow.

## Overrides

### PR title validation

| Override | Default |
| -------- | ------- |
| `enable-pr-title-verify` | `true` |
| `pull-request-title` | `` |
| `pull-request-allowed-prefixes` | `` |
| `pull-request-min-length-title` | `` |
| `pull-request-max-length-title` | `` |
| `pull-request-case-sensitive-prefix` | `` |

To disable the PR title validation

```yaml
jobs:
  pull-request-checks:
    with:
      enable-pr-title-verify: false
```

> [!TIP]
> See the [validate-pr-title composite action
> docs](../.github/actions/validate-pr-title/README.md) for more info on the
> other overrides.

### Automatic merge of Dependabot PRs

| Override | Default |
| -------- | ------- |
| `enable-auto-merge-dependabot` | `false` |
| `auto-merge-types` | `version-update:semver-patch;version-update:semver-minor` |

To enable automatic merge of Dependabot PRs

```yaml
jobs:
  pull-request-checks:
    with:
      enable-auto-merge-dependabot: true
```

> [!TIP]
> Supported values for `auto-merge-types` can be found under `update-types` in
> [Dependabot GitHub docs](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference#ignore--)
