# PR checks usage docs (library)

These docs explain how to use our reusable workflow for PRs in app repos that
produce Maven library packages

- [PR checks for libraries](../.github/workflows/ci-pr-checks-lib.yml)

This workflow support

- PR title validation
- Package build and scan
- Automatic merge of Dependabot PRs

## Usage

In your application repository, create a new file called
`.github/workflows/pull-request.yml` (best practice naming)

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
    uses: felleslosninger/github-workflows/.github/workflows/ci-pr-checks-lib.yml@main
    permissions:
      contents: write
      pull-requests: write
    secrets: inherit
```

> [!NOTE]
> The workflow trigger types are needed to also trigger on `edited`, which
> ensures that the workflow runs when the PR title is updated (in the case of PR
> title validation failures).

You do not need to override anything to start using this workflow.

## Overrides

### PR title validation

This is handled via a separate workflow job called `validate-pr-title`.

To disable the PR title validation

```yaml
jobs:
  pull-request-checks:
    with:
      enable-pr-title-verify: false
```

#### PR title validation overrides

| Override | Workflow default |
| -------- | ---------------- |
| `pull-request-title` | `` |
| `pull-request-allowed-prefixes` | `` |
| `pull-request-min-length-title` | `` |
| `pull-request-max-length-title` | `` |
| `pull-request-case-sensitive-prefix` | `` |

> [!TIP]
> See the [validate-pr-title composite action
> docs](../.github/actions/validate-pr-title/README.md) on override usage.

### Package build and scan

This is handled via a separate workflow job called `build-and-test-java`.

#### Java setup overrides

| Override | Workflow default |
| -------- | ---------------- |
| `java-version` | `25` |
| `java-distribution` | `liberica` |
| `cache-path` | `**/pom.xml` |

#### Maven overrides

| Override | Workflow default |
| -------- | ---------------- |
| `maven-lifecycle` | `install` |
| `maven-clean` | `false` |
| `maven-update-snapshots` | `false` |

#### Trivy overrides

| Override | Workflow default |
| -------- | ---------------- |
| `trivy-library-disable-scan` | `false` |
| `trivy-library-ignore-unfixed` | `true` |
| `trivy-library-severity` | `HIGH,CRITICAL` |
| `trivy-os-disable-scan` | `false` |
| `trivy-os-ignore-unfixed` | `true` |
| `trivy-os-severity` | `CRITICAL` |
| `trivy-version` | `` |

> [!TIP]
> See the [trivy-scan composite action
> docs](../.github/actions/trivy-scan/README.md) for the other overrides.

#### Artifact upload overrides

| Override | Workflow default |
| -------- | ---------------- |
| `artifact-name` | `` |
| `artifact-path` | `` |

You need to override both to enable artifact upload.

### Automatic merge of Dependabot PRs

This is handled via a separate workflow job called `call-auto-merge`.

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
