# Pull request usage docs (library)

These docs explain how to use our reusable workflow for PRs in app repos that
produce Maven library packages

- [PR checks for libraries](../.github/workflows/ci-pr-checks-lib.yml)

This workflow is split into the following jobs

- PR title validation
- Package build and scan
- Automatic merge of Dependabot PR

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

This is handled via a separate workflow job called `validate-pr-title`. This is
enabled by default.

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
> See the [validate-pr-title composite
> action](../.github/actions/validate-pr-title/README.md) for more docs on the
> overrides.

### Package build and scan

This is handled via a separate workflow job called `build-and-test-java`. This
will always run.

#### Package build and scan overrides

##### Java setup

| Override | Workflow default |
| -------- | ---------------- |
| `java-version` | `25` |
| `java-distribution` | `liberica` |
| `cache-path` | `**/pom.xml` |

##### Maven

| Override | Workflow default |
| -------- | ---------------- |
| `maven-lifecycle` | `install` |
| `maven-clean` | `false` |
| `maven-update-snapshots` | `false` |

##### Trivy

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
> See the [trivy-scan composite action](../.github/actions/trivy-scan/README.md)
> for more docs on the overrides.

##### Artifact upload

| Override | Workflow default |
| -------- | ---------------- |
| `artifact-name` | `` |
| `artifact-path` | `` |

> [!NOTE]
> You need to override both `artifact-name` and `artifact-path` to enable
> artifact upload.

### Automatic merge of Dependabot PR

This is handled via a separate workflow job called `call-auto-merge`. This is
disabled by default.

To enable automatic merge of Dependabot PRs

```yaml
jobs:
  pull-request-checks:
    with:
      enable-auto-merge-dependabot: true
```

#### Automatic merge of Dependabot PR overrides

| Override | Default |
| -------- | ------- |
| `auto-merge-types` | `version-update:semver-patch;version-update:semver-minor` |

> [!TIP]
> Supported values for `auto-merge-types` can be found under `update-types` in
> the [Dependabot GitHub
> docs](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference#ignore--).
