# Pull request usage docs (image)

These docs explain how to use our reusable workflow for PRs in app repos that
produce container images

- [PR checks for container images](../.github/workflows/ci-pr-checks-image.yml)

This workflow is split into the following jobs

- PR title validation
- Container image scanning (based on application type)
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
    uses: felleslosninger/github-workflows/.github/workflows/ci-pr-checks-image.yml@main
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
> See the [validate-pr-title composite
> action](../.github/actions/validate-pr-title/README.md) for more docs on the
> overrides.

### Container image scanning

This is handled via 3 separate workflow jobs depending on application type

- `call-spring-boot-container-scan`
- `call-quarkus-container-scan`
- `call-docker-container-scan`

To disable container scanning

```yaml
jobs:
  pull-request-checks:
    with:
      enable-container-image-scan: false
```

The 3 jobs support some common overrides, as well as overrides based on the
application type.

#### Container image scanning (common overrides)

##### Application type

| Override | Workflow default |
| -------- | ---------------- |
| `application-type` | `spring-boot` |

> [!NOTE]
> Valid values are `spring-boot`, `quarkus` or `docker`.

##### Image metadata

| Override | Workflow default |
| -------- | ---------------- |
| `image-name` | `` |

##### Application path

| Override | Workflow default |
| -------- | ---------------- |
| `application-path` | `./` |

##### Base image

These are applicable to Spring Boot and Quarkus application types.

| Override | Workflow default |
| -------- | ---------------- |
| `image-pack` | `builder-noble-java-tiny` |
| `image-pack-tag` | `latest` |

> [!TIP]
> See the [Paketo builder image
> docs](https://paketo.io/docs/reference/builders-reference/) for choices. Valid
> tag overrides can be found from the GitHub releases of the image pack in use
> ([example](https://github.com/paketo-buildpacks/builder-noble-java-tiny/releases)).

##### Java setup

These are applicable to Spring Boot and Quarkus application types.

| Override | Workflow default |
| -------- | ---------------- |
| `java-version` | `25` |
| `java-distribution` | `liberica` |
| `cache-path` | `**/pom.xml` |

> [!NOTE]
> Java version must be overridden if the application differs from the default.
> The Java distribution should only be overridden if there are issues with our
> current default.

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

#### Container image scanning (Spring Boot overrides)

| Override | Workflow default |
| -------- | ---------------- |
| `maven-lifecycle` | `install` |
| `maven-skip-tests` | `true` |
| `module-name` | `` |

#### Container image scanning (Quarkus overrides)

| Override | Workflow default |
| -------- | ---------------- |
| `native` | `false` |

#### Container image scanning (Docker overrides)

| Override | Workflow default |
| -------- | ---------------- |
| `docker-build-context` | `.` |
| `add-git-package-token` | `false` |

### Automatic merge of Dependabot PR

This is handled via a separate workflow job called `call-auto-merge`.

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
> [Dependabot GitHub docs](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference#ignore--)
