# Library release usage docs

These docs explain how to use our reusable workflow for GitHub artifact releases
in library repos

- [Build and publish library](../.github/workflows/ci-build-publish-lib.yml)

This workflow is split into the following jobs

- Write inputs to step summary (no overrides supported)
- Build and publish package

## Usage

In your application repository, create a new file called
`.github/workflows/release.yml` (best practice naming)

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
> See examples section on how to use this for multi-module library repos.

## Overrides

Overrides are only supported for the build/publish library workflow job.

### Build and publish library

This is handled via a separate workflow job called `build-publish-package`. This
will always run.

#### Build and publish library overrides

##### Fetch depth

| Override | Workflow default |
| -------- | ---------------- |
| `fetch-depth` | `1` |

> [!NOTE]
> Only a single commit is fetched by default, for the ref/SHA that triggered the
> workflow. Set fetch depth to 0 to fetch all history for all branches and tags.
> This is rarely necessary.

##### Java setup

| Override | Workflow default |
| -------- | ---------------- |
| `java-version` | `25` |
| `java-distribution` | `liberica` |
| `cache-path` | `**/pom.xml` |

> [!NOTE]
> Java version must be overridden if the application differs from the default.
> The Java distribution should only be overridden if there are issues with our
> current default.

##### Package version

| Override | Workflow default |
| -------- | ---------------- |
| `package-version` | Not set |

> [!NOTE]
> When this is overridden, `mvn versions set` is run using the overridden
> package version.

##### Maven

| Override | Workflow default |
| -------- | ---------------- |
| `maven-profile` | Not set |
| `deployment-repository` | Not Set |
| `module-name` | Not Set |

##### Trivy

| Override | Workflow default |
| -------- | ---------------- |
| `trivy-library-disable-scan` | `false` |
| `trivy-library-ignore-unfixed` | `true` |
| `trivy-library-severity` | `HIGH,CRITICAL` |
| `trivy-version` | Not set |

> [!TIP]
> See the [trivy-scan composite action](../.github/actions/trivy-scan/README.md)
> for more docs on the overrides.

##### SBOM

| Override | Workflow default |
| -------- | ---------------- |
| `sbom-artifact-id` | Not set |
| `sbom-path` | Not set |

> [!NOTE]
> Leaving `sbom-artifact-id` empty will default this to the GitHub repository
> name. Leaving `sbom-path` empty will default this to the repo root.

## Examples

### Multi-module library repos

TBA
