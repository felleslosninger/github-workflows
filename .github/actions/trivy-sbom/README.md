# GitHub Action: Trivy SBOM generation

Author: **Digdir Platform Team**

## Description

This GitHub Action generates and uploads a Software Bill of Materials (SBOM)
using Trivy. It supports both container images and filesystem paths (e.g., for
library repositories). By generating an SBOM, we create a formal,
machine-readable inventory of all software components and dependencies used in
our artifacts, which is essential for security auditing and compliance.

By default, this action:

- **Standardizes Naming:** Sanitizes the `artifact-id` (replacing `/`, `.`, and spaces with `-`) and constructs filenames following the pattern `sbom-name-version.json` (or `.spdx`).
- **Handles Dynamic Versioning:** If no `version` is provided, it automatically falls back to the GitHub Run ID and Attempt (`run_number`-`run_attempt`) to ensure unique, traceable filenames.
- **Integrates with Dependency Graph:** Defaults to the `github` format, allowing for native integration with the GitHub Enterprise Dependency Graph to populate the dependency list for the repository.
- **Optimizes Performance:** Supports skipping Trivy setup (`skip-setup`) if a vulnerability scan has already been performed in the same job, saving compute time and avoiding redundant database downloads.
- **Enables Browser Viewing:** Utilizes `actions/upload-artifact@v7.0.1` with archiving disabled (`archive: false`), allowing developers to view the SBOM JSON or SPDX content directly in the GitHub Actions summary without downloading a zip file.

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
| `format` | Output format ('github', 'cyclonedx', 'spdx', 'spdx-json') | false | `github` |
| `upload-artifact` | Whether to automatically upload the SBOM as a GitHub artifact | false | `true` |
| `skip-setup` | Skip Trivy setup if it has already been run in a previous step | false | `false` |

## Example usage

Workflow using image builds

```yaml
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Generate SBOM for image
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
      - name: Generate SBOM for library
        uses: felleslosninger/github-workflows/.github/actions/trivy-sbom@main
        with:
          scan-type: 'fs'
          artifact-id: my-shared-library
          format: 'spdx'
```
