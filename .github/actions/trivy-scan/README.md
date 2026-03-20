# GitHub Action: Trivy scan

Author: **Digdir Platform Team**

## Description

This GitHub Action performs scanning using Trivy on container images or
filesystem paths. It supports both library and OS vulnerability scans with
configurable severity levels and can respect `.trivyignore` files for excluding
specific vulnerabilities. The Trivy version to use is possible to override, but
it will use the default from the official Trivy Action if unset. We add a
warning to the step summary if the number of ignored vulnerabilities in
`.trivyignore` exceeds what the Platform team considers a high amount. By
default, it will also scan for secrets that might have been accidentally
included in plaintext. We try to only fail the scans when there is actionable
feedback (like library scan failures for fixable CVEs). We try to avoid failing
the pipeline when there is nothing you can do about it, like a OS vulnerability
in the `latest` tag of our base image.

By default we

- Split library and OS scanning so that we can decide severity thresholds
  individually for the different scans
- Ignore unfixed vulnerabilities by default to avoid stopping the pipeline when
  there's no fix available yet (but this can be overridden)
- We hide the full output from the Trivy scan, and move it out to the step
  summary if there's something you should look at (but this can be overridden)
- Scan for plaintext secrets
- Disable the noisy VEX notice that is included in scan results

## Prerequisites

None

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `scan-type` | Type of scan to run ('image' or 'fs') | false | `image` |
| `image-ref` | Image reference to scan (implicitly required if `scan-type` is `image`) | false | `""` |
| `application-path` | Path to application for Trivy scan (to detect .trivyignore) | false | `./` |
| `library-disable-scan` | Disable library scan | false | `false` |
| `library-ignore-unfixed` | Ignore unfixed vulnerabilities in library scan | false | `true` |
| `library-severity` | When to fail the scan with library vulnerabilities | false | `HIGH,CRITICAL` |
| `library-hide-progress` | Hide progress output for library scan | false | `true` |
| `os-disable-scan` | Disable OS scan | false | `false` |
| `os-ignore-unfixed` | Ignore unfixed vulnerabilities in OS scan | false | `true` |
| `os-severity` | When to fail the scan with OS vulnerabilities | false | `CRITICAL` |
| `os-hide-progress` | Hide progress output for OS scan | false | `true` |
| `os-exit-code` | Exit code when OS vulnerabilities are found (0 = informational, 1 = blocking) | false | `"0"` |
| `trivy-version` | Version of Trivy to use for scanning | false | `""` |
| `trivyignore-warning-threshold` | Number of ignored vulnerabilities before we issue a warning | false | `10` |

## Example usage

Workflow using image builds

```yaml
jobs:
  some-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run Trivy vulnerability scanner
        uses: felleslosninger/github-workflows/.github/actions/trivy-scan@main
        with:
          image-ref: my-registry/my-image:latest
```

Workflow using file scan (usually pull request workflows to catch vulns early)

```yaml
jobs:
  some-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run Trivy vulnerability scanner
        uses: felleslosninger/github-workflows/.github/actions/trivy-scan@main
        with:
          scan-type: 'fs'
```

## Upgrade

Dependabot will create relevant PRs for [Trivy
Action](https://github.com/aquasecurity/trivy-action/releases) upgrades. We use
the default version of Trivy from the upstream Trivy Action, but you can
override this if the official action have not yet upgraded to a version of Trivy
we want to use.

## How it works

This action utilizes a composite run to perform Trivy vulnerability scanning. It
validates inputs, checks for `.trivyignore` files, runs library scans (for both
image and fs types), and OS scans (for image type only). Results are published
to the GitHub step summary with expandable details.
