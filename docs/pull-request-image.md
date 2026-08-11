# Pull request usage docs (image)

These docs explain how to use our reusable workflow for PRs in app repos that
produce container images

- [PR checks for container images](../.github/workflows/ci-pr-checks-image.yml)

This workflow is split into the following jobs

- PR title validation
- Container image scanning (based on application type)
- Automatic merge of Dependabot PRs

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
    uses: felleslosninger/github-workflows/.github/workflows/ci-pr-checks-image.yml@main
    permissions:
      contents: write
      pull-requests: write
    secrets: inherit
```

> [!NOTE]
>
> - **Trigger types:** The workflow trigger types are needed to also trigger on
>   `edited`, which ensures that the workflow runs when the PR title is updated
>   (in the case of PR title validation failures).
>
> - **Supported repository type:** Because this configuration uses all the
>   workflow defaults, it assumes the repository is a **single-module Spring
>   Boot application** located at the repository root. It expects a standard
>   Maven setup with a `pom.xml` in the root directory, built using **Java 25**.
>   If your repository is a multi-module project, a monorepo, a Quarkus/Docker
>   application, or uses a different Java version, you must provide the
>   necessary overrides. See the [**Examples**](#examples) section below for how
>   to configure these specific setups.

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

### Container image scanning

This is handled via 3 separate workflow jobs depending on application type

- `call-spring-boot-container-scan`
- `call-quarkus-container-scan`
- `call-docker-container-scan`

This is enabled by default.

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

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `application-type` | `string` | `spring-boot` |

> [!NOTE]
> Valid values are `spring-boot`, `quarkus` or `docker`.

##### Image metadata

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `image-name` | `string` | Not set |

> [!TIP]
> If `image-name` is not set, the workflow will automatically derive a name
> (typically based on your repository name). You only need to override this if
> you have a specific naming convention or are building multiple distinct images
> from a single monorepo.

##### Application path

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `application-path` | `string` | `./` |

> [!NOTE]
> **Path formatting:** The path is relative to the repository root. If you
> override this value, you **must include a trailing slash** (e.g., `backend/`).
> The underlying build scripts concatenate this directly with filenames
> (evaluating to `${APP_PATH}pom.xml`), and the build will fail if the slash is
> missing.

##### Base image

These are applicable to Spring Boot and Quarkus application types.

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `image-pack` | `string` | `builder-noble-java-tiny` |
| `image-pack-tag` | `string` | `latest` |

> [!TIP]
> See the [Paketo builder image
> docs](https://paketo.io/docs/reference/builders-reference/) for choices. Valid
> tag overrides can be found from the GitHub releases of the image pack in use
> ([example](https://github.com/paketo-buildpacks/builder-noble-java-tiny/releases)).

##### Java setup

These are applicable to Spring Boot and Quarkus application types.

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `java-version` | `string` | `25` |
| `java-distribution` | `string` | `liberica` |
| `cache-path` | `string` | `**/pom.xml` |

> [!NOTE]
> Java version must be overridden if the application differs from the default.
> The Java distribution should only be overridden if there are issues with our
> current default.

##### Trivy

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `trivy-library-disable-scan` | `boolean` | `false` |
| `trivy-library-ignore-unfixed` | `boolean` | `true` |
| `trivy-library-severity` | `string` | `HIGH,CRITICAL` |
| `trivy-os-disable-scan` | `boolean` | `false` |
| `trivy-os-ignore-unfixed` | `boolean` | `true` |
| `trivy-os-severity` | `string` | `CRITICAL` |
| `trivy-version` | `string` | Not set |

> [!TIP]
> See the [trivy-scan composite action](../.github/actions/trivy-scan/README.md)
> for more docs on the overrides.

#### Container image scanning (Spring Boot overrides)

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `maven-lifecycle` | `string` | `install` |
| `maven-skip-tests` | `boolean` | `true` |
| `module-name` | `string` | Not set |

> [!NOTE]
> **Multi-module projects:** If your application is part of a multi-module Maven
> project, you should override `module-name` instead of `application-path`. The
> workflow will automatically append the `-pl <module-name> -am` flags to ensure
> required internal dependencies are built before generating the image.

#### Container image scanning (Quarkus overrides)

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `native` | `boolean` | `false` |

> [!NOTE]
> Setting `native` to `true` compiles a GraalVM native image using Paketo
> buildpacks. Be aware that native compilation is highly resource-intensive and
> will significantly increase the duration of your PR validation checks.

#### Container image scanning (Docker overrides)

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `docker-build-context` | `string` | `.` |
| `add-git-package-token` | `boolean` | `false` |

> [!NOTE]
> **Dockerfile path resolution:** By default, the workflow looks for a
> `Dockerfile` inside the specified `application-path`. However, if you set
> `add-git-package-token` to `true`, the workflow mirrors the behavior of our
> publish pipelines and will *strictly* look for the Dockerfile at
> `docker/Dockerfile` relative to the repository root.

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
| `auto-merge-types` | `string` | `version-update:semver-patch;version-update:semver-minor` |

> [!TIP]
>
> - **Configuration details:** Supported values for `auto-merge-types` can be
>   found under `update-types` in the [Dependabot GitHub
>   docs](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference#ignore--).
>
> - **Security note:** For supply-chain security reasons, automatically merged
>   Dependabot PRs do *not* automatically trigger a new container image
>   build/deployment. This ensures infrastructure updates require human
>   oversight before rolling out.

## Examples

### Multi-module Maven project

Use this setup when your repository has a root `pom.xml` and multiple
interconnected modules (e.g., `domain`, `common`, `api`), where only one module
is the actual bootable application.

By using `module-name`, the workflow knows to build the entire project reactor
so internal dependencies are resolved correctly. We leave `cache-path` at its
default (`**/pom.xml`) so the cache correctly invalidates if you update a
dependency in a shared library module.

**`.github/workflows/pull-request.yml`**

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
      # Triggers 'mvn ... -pl "api" -am' to build internal dependencies first
      module-name: 'api'
    secrets: inherit
```

### Monorepo with independent applications

Use this setup when your repository contains completely separate applications
that do *not* share a root `pom.xml` or internal dependencies.

For monorepos, you should create a separate workflow file for each application
and use the `paths` trigger. This ensures that changes to `Application A` do not
unnecessarily trigger the build, Trivy scans, and consume GitHub Actions minutes
for `Application B`.

**`.github/workflows/pr-backend-service.yml`**

```yaml
name: PR - Backend Service

on:
  pull_request:
    types:
      - opened
      - reopened
      - synchronize
      - edited
    branches:
      - main
    paths:
      - 'apps/backend-service/**'
      - '.github/workflows/pr-backend-service.yml'

jobs:
  pull-request-checks:
    uses: felleslosninger/github-workflows/.github/workflows/ci-pr-checks-image.yml@main
    permissions:
      contents: write
      pull-requests: write
    with:
      # Target the specific directory for the build
      application-path: "apps/backend-service/"
      # Isolate the cache to this specific app
      cache-path: "apps/backend-service/pom.xml"
    secrets: inherit
```

**`.github/workflows/pr-auth-service.yml`**

```yaml
name: PR - Auth Service

on:
  pull_request:
    types:
      - opened
      - reopened
      - synchronize
      - edited
    branches:
      - main
    paths:
      - 'apps/auth-service/**'
      - '.github/workflows/pr-auth-service.yml'

jobs:
  pull-request-checks:
    uses: felleslosninger/github-workflows/.github/workflows/ci-pr-checks-image.yml@main
    permissions:
      contents: write
      pull-requests: write
    with:
      # Target the specific directory for the build
      application-path: "apps/auth-service/"
      # Isolate the cache to this specific app
      cache-path: "apps/auth-service/pom.xml"
    secrets: inherit
```

> [!CAUTION]
> **Path formatting:** When overriding `application-path`, you **must** include
> the trailing slash (e.g., `apps/backend-service/`). The underlying scripts
> concatenate this directly (evaluating to `${APP_PATH}pom.xml`), and the build
> will fail if the slash is omitted.
