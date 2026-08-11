# Pull request usage docs (library)

These docs explain how to use our reusable workflow for PRs in app repos that
produce Maven library packages

- [PR checks for libraries](../.github/workflows/ci-pr-checks-lib.yml)

This workflow is split into the following jobs

- PR title validation
- Package build and scan
- Automatic merge of Dependabot PR

## GitHub workflow permissions

This is the list of the current workflow permissions used as part of these
workflows.

| Permission | Purpose |
| ---------- | ------- |
| `contents: write` | Needed for merging PRs via Dependabot auto-merge functionality |
| `pull-requests: write` | Needed for modifying PRs via Dependabot auto-merge functionality |

## GitHub secrets

This is the list of the current GitHub secrets used as part of this workflow.

| Secret | Type | Visibility | Admin action required |
| ------ | ---- | ---------- | --------------------- |
| `GH_PACKAGES_READ_USER` | Org. secret | All repositories | No |
| `GH_PACKAGES_READ_PAT` | Org. secret | All repositories | No |

> [!NOTE]
> No GitHub admin action is required to use this workflow.

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

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `pull-request-title` | `string` | Not set |
| `pull-request-allowed-prefixes` | `string` | Not set |
| `pull-request-min-length-title` | `string` | Not set |
| `pull-request-max-length-title` | `string` | Not set |
| `pull-request-case-sensitive-prefix` | `string` | Not set |

> [!TIP]
> See the [validate-pr-title composite
> action](../.github/actions/validate-pr-title/README.md) for more docs on the
> overrides.

### Package build and scan

This is handled via a separate workflow job called `build-and-test-java`. This
will always run.

#### Package build and scan overrides

##### Java setup

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `java-version` | `string` | `25` |
| `java-distribution` | `string` | `liberica` |
| `cache-path` | `string` | `**/pom.xml` |

> [!NOTE]
> Java version must be overridden if the application differs from the default.
> The Java distribution should only be overridden if there are issues with our
> current default.

##### Maven

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `maven-lifecycle` | `string` | `install` |
| `maven-clean` | `boolean` | `false` |
| `maven-update-snapshots` | `boolean` | `false` |

> [!NOTE]
> **Lifecycle validation:** The `maven-lifecycle` input only accepts `test`,
> `package`, `verify`, or `install` (default). `install` is required by default
> to populate the local `.m2` repository for the Trivy offline scan.
> **Build optimization:** Because runners are ephemeral, `maven-clean` defaults
> to `false` to save time. Only enable `maven-update-snapshots` if your PR
> depends on newly published internal snapshots.

##### Trivy

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `application-path` | `string` | `./` |
| `trivy-library-disable-scan` | `boolean` | `false` |
| `trivy-library-ignore-unfixed` | `boolean` | `true` |
| `trivy-library-severity` | `string` | `HIGH,CRITICAL` |
| `trivy-version` | `string` | Not set |

> [!TIP]
> See the [trivy-scan composite action](../.github/actions/trivy-scan/README.md)
> for more docs on the overrides.

##### Artifact upload

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `artifact-name` | `string` | Not set |
| `artifact-path` | `string` | Not set |

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

| Override | Type | Default |
| -------- | ---- | ------- |
| `auto-merge-types` | `string` | Not set |

> [!TIP]
>
> - **Configuration details:** Supported values for `auto-merge-types` can be
>   found under `update-types` in the [Dependabot GitHub
>   docs](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference#ignore--).
>
> - **Security note:** For supply-chain security reasons, automatically merged
>   Dependabot PRs do *not* trigger any library publish/release workflows.
>   This ensures infrastructure updates require human oversight before rolling
>   out.
