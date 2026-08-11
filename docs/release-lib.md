# Library release usage docs

These docs explain how to use our reusable workflow for GitHub artifact releases
in library repos.

- [Build and publish library](../.github/workflows/ci-build-publish-lib.yml)

This workflow is split into the following jobs:

- Write inputs to step summary (no overrides supported)
- Build and publish package

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
`.github/workflows/release.yml` (best practice naming).

```yaml
name: Release

on:
  release:
    types:
      - created

jobs:
  create-release:
    uses: felleslosninger/github-workflows/.github/workflows/ci-build-publish-lib.yml@main
    permissions:
      contents: read
      packages: write
    with:
      package-version: ${{ github.event.release.tag_name }}
      deployment-repository: ${{ github.repository }}
    secrets: inherit
```

> [!NOTE]
>
> - **Standard flow:** This workflow runs `mvn clean install` to resolve and
>   build the full dependency tree locally before running the Trivy filesystem
>   scan. This ensures accurate CVE detection across transitively resolved
>   internal dependencies.
>
> - **Supported repository type:** Because this configuration uses all workflow
>   defaults, it assumes the repository is a **single-module Java library**
>   located at the repository root, built using **Java 25**. See the
>   [**Examples**](#examples) section below for how to release multi-module
>   libraries.

## Overrides

Overrides are only supported for the build/publish library workflow job.

### Build and publish library

This is handled via a separate workflow job called `build-publish-package`. This
will always run.

#### Build and publish library overrides

##### Fetch depth

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `fetch-depth` | `number` | `1` |

> [!NOTE]
> Only a single commit is fetched by default, for the ref/SHA that triggered the
> workflow. Set fetch depth to `0` to fetch all history for all branches and
> tags. This is rarely necessary.

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

##### Package version

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `package-version` | `string` | Not set |

> [!NOTE]
> When this is overridden, `mvn versions:set` is run dynamically during the
> build using the overridden package version before packaging the artifact.

##### Maven

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `maven-profile` | `string` | Not set |
| `deployment-repository` | `string` | Not set |
| `module-name` | `string` | Not set |

> [!CAUTION]
> **Path formatting:** If using `module-name` for multi-module projects, this
> **must** be the directory path of the module relative to the reactor root
> (e.g., `payment/api`), NOT the Maven coordinate form (`:artifactId` or
> `groupId:artifactId`). The Trivy scanner and SBOM generators derive their
> filesystem target paths directly from this input.

##### Trivy

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `trivy-library-disable-scan` | `boolean` | `false` |
| `trivy-library-ignore-unfixed` | `boolean` | `true` |
| `trivy-library-severity` | `string` | `HIGH,CRITICAL` |
| `trivy-version` | `string` | Not set |

> [!TIP]
> See the [trivy-scan composite action](../.github/actions/trivy-scan/README.md)
> for more docs on the overrides.

##### SBOM

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `sbom-artifact-id` | `string` | Not set |
| `sbom-path` | `string` | Not set |

> [!NOTE]
>
> - Leaving `sbom-path` empty will default to the `module-name` if provided, or
>   the repository root if not.
>
> - Leaving `sbom-artifact-id` empty will default to the clean `sbom-path`
>   folder name (for multi-module projects), or the GitHub repository name (for
>   root projects).

## Examples

### Multi-module library repos

When working with a multi-module Maven project where you want to publish
specific sub-modules independently, use GitHub Actions' `matrix` strategy
combined with the `module-name` input.

By setting `module-name`, the workflow runs `mvn install -pl <module-name> -am`.
This ensures that any upstream reactor dependencies (like a shared `common`
module) are built in the correct order locally before packaging and publishing
the target module.

```yaml
name: Release modules

on:
  release:
    types:
      - created

jobs:
  publish-modules:
    strategy:
      matrix:
        # These MUST be the directory paths, not Maven coordinates
        module: ['libs/common', 'libs/api-client']

    uses: felleslosninger/github-workflows/.github/workflows/ci-build-publish-lib.yml@main
    permissions:
      contents: read
      packages: write
    with:
      package-version: ${{ github.event.release.tag_name }}
      deployment-repository: ${{ github.repository }}
      module-name: ${{ matrix.module }}
    secrets: inherit
```
