# GitHub Action: Trivy SBOM generation

Author: **Digdir Platform Team**

## Description

This GitHub Action generates and uploads a Software Bill of Materials (SBOM)
using Trivy. It supports both container images and filesystem paths (e.g., for
library repositories). By generating an SBOM, we create a formal,
machine-readable inventory of all software components and dependencies used in
our artifacts, which is essential for security auditing and compliance. The
result is both added as an artifact, and uploaded to the GitHub Dependency
Graph.

By default, this action:

- **Standardizes naming:** Sanitizes the `artifact-id` (replacing `/`, `.`, and
  spaces with `-`) and constructs filenames following the pattern
  `sbom-name-version.json`
- **Handles dynamic versioning:** If no `version` is provided, it automatically
  falls back to the GitHub Run ID and Attempt (`run_number`-`run_attempt`)
- **Integrates with Dependency Graph:** Hardcoded to the `github` format,
  allowing for native integration with the GitHub Enterprise Dependency Graph to
  populate the dependency list for the repository
- **Enables browser viewing:** Utilizes `actions/upload-artifact` with archiving
  disabled (`archive: false`), allowing developers to view the SBOM JSON content
  directly in the GitHub Actions summary without downloading a zip file

## Prerequisites

None

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `scan-type` | Type of scan to run ('image' or 'fs') | false | `image` |
| `artifact-id` | Unique ID for the artifact (e.g., the app name) | **true** | - |
| `image-ref` | Image reference to scan (required if `scan-type` is `image`) | false | `""` |
| `version` | The version or tag of the artifact (falls back to run info if empty) | false | `""` |
| `application-path` | Path to scan (required if `scan-type` is `fs`) | false | `.` |
| `upload-artifact` | Whether to automatically upload the SBOM as a GitHub artifact | false | `true` |
| `skip-setup` | Skip Trivy setup if it has already been run in a previous step | false | `false` |

## Example usage

Workflow using image builds

```yaml
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Run Trivy SBOM generation
        uses: felleslosninger/github-workflows/.github/actions/trivy-sbom@main
        with:
          artifact-id: my-application
          image-ref: ghcr.io/digdir/my-application:v1.2.3
          version: v1.2.3
```

Workflow using libraries

```yaml
jobs:
  library-release:
    runs-on: ubuntu-latest
    steps:
      - name: Run Trivy SBOM generation
        uses: felleslosninger/github-workflows/.github/actions/trivy-sbom@main
        with:
          scan-type: fs
          artifact-id: my-maven-library
          version: v1.2.3
```

Note that if Trivy scan composite action has been run before this step, you can
skip the Trivy setup by adding `skip-setup: true`.
