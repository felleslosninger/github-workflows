# github-workflows

## Overview

This repository contains reusable GitHub Workflows and composite Actions for
common CI/CD tasks. Workflows are organized by purpose and application type.

## Composite actions

We have a set of composite actions that contain reusable steps to avoid
duplication, and to reduce maintenance for the Platform team

- ARCHIVED: [acr-login](./.github/actions/archived-actions/acr-login/): This composite action
  is designed to login to Azure Container Registry via federated credentials
- [cr-login](./.github/actions/cr-login/): This composite action is designed
  to login to Azure Container Registry via federated credentials, or GitHub
  Container Registry via GitHub token
- [docker-push-image](./.github/actions/docker-push-image/): This composite
  action is designed to push a Docker image to a container registry and extract
  its digest
- [image-metadata](./.github/actions/image-metadata): This composite action
  generates Docker image metadata for workflows that need a consistent image
  name and tag
- [image-signing](./.github/actions/image-signing): This composite action is
  designed to sign our container images using keyless signing with Cosign,
  Fulcio CA and Rekor transparency log
- [json-to-summary](./.github/actions/json-to-summary): This composite action is
  designed to write JSON content to the step summary
- [sh-scan](./.github/actions/sh-scan): This composite action is designed to
  scan for indicators of compromise (IOCs) related to Shai-Hulud supply chain
  attacks in NPM
- [trivy-sbom](./.github/actions/trivy-sbom): This composite action is designed
  to generate and upload a Software Bill of Materials (SBOM) using Trivy
- [trivy-scan](./.github/actions/trivy-scan): This composite action is designed
  to perform scanning using Trivy on container images or filesystem paths
- [validate-pr-title](./.github/actions/validate-pr-title): This composite
  action is designed to validate a PR title according to FEL best practices
- [zap-scan](./.github/actions/zap-scan): This composite action is designed to
  perform ZAP (OWASP Zed Attack Proxy) Dynamic Application Security Testing
  (DAST) scanning

These actions are used as part of our golden path workflows, but can also be
included in custom workflows depending on your needs. Check out the READMEs in
the corresponding action directories for more information on how to use them.

## Golden path workflows (recommended)

For new projects, you should use the following workflows depending on your needs.

### PR Checks

#### PR Checks overview

For PRs to main, use the golden path workflow that matches your project type

- [ci-pr-checks-lib.yml](.github/workflows/ci-pr-checks-lib.yml): Golden path
  for PRs in Maven library projects (no container image). Verifies the PR title,
  builds and tests with Maven, runs a Trivy filesystem scan, optionally uploads
  a build artifact, and can auto-merge Dependabot PRs.
- [ci-pr-checks-image.yml](.github/workflows/ci-pr-checks-image.yml): Golden
  path for PRs in containerized applications. Verifies the PR title, then
  delegates to the application-specific container build-and-scan workflow based
  on the `application-type` input, and can auto-merge Dependabot PRs.

For containerized applications, `ci-pr-checks-image.yml` runs one of the
following application-specific workflows

- [ci-spring-boot-container-scan.yml](.github/workflows/ci-spring-boot-container-scan.yml):
  Builds and scans temporary Spring Boot container images
- [ci-quarkus-container-scan.yml](.github/workflows/ci-quarkus-container-scan.yml):
  Builds and scans temporary Quarkus JVM or GraalVM native container images
- [ci-docker-container-scan.yml](.github/workflows/ci-docker-container-scan.yml):
  Builds and scans temporary custom Docker images

Both `ci-pr-checks-image.yml` and `ci-pr-checks-lib.yml` optionally run the
Dependabot auto-merge workflow

- [misc-approve-and-merge-dependabot-pr.yml](.github/workflows/misc-approve-and-merge-dependabot-pr.yml):
  Auto-approves and merges supported Dependabot updates

#### PR Checks usage docs

- [PR checks (image)](./docs/pull-request-image.md)
- [PR checks (library)](./docs/pull-request-lib.md)

### Build and publish container images

#### Build and publish container images overview

[ci-build-publish-image.yml](.github/workflows/ci-build-publish-image.yml):
Proxy workflow for building and publishing images after commits to main.
Automatically triggers the appropriate workflow (Spring Boot, Quarkus, or custom
Docker) based on application type, with Trivy vulnerability scanning and
optional image signing (default `true`). This workflow will trigger one of the
following application-specific workflows

- [ci-spring-boot-build-publish-image.yml](.github/workflows/ci-spring-boot-build-publish-image.yml):
  Builds and publishes Spring Boot container images using Paketo buildpacks
- [ci-quarkus-build-publish-image.yml](.github/workflows/ci-quarkus-build-publish-image.yml):
  Builds Quarkus JVM or GraalVM native images
- [ci-docker-build-publish-image.yml](.github/workflows/ci-docker-build-publish-image.yml):
  Generic Docker image builds from Dockerfile

#### Build and publish container usage docs

- [Deployment (image)](./docs/deployment-image.md)

### Build and publish Maven libraries

#### Build and publish Maven libraries overview

Workflows for building and publishing Maven libraries after commits to main

- [ci-build-publish-lib.yml](.github/workflows/ci-build-publish-lib.yml): The
  golden path for building and publishing Maven libraries. Use this for all
  projects, with or without internal Maven dependencies

#### Build and publish Maven libraries usage docs

- [Release (library)](./docs/release-lib.md)

## Other utility workflows

### Other utility workflows overview

- [lint.yml](.github/workflows/lint.yml): Validates workflow files with
  actionlint and Markdown docs with markdownlint (local workflow)
- [on-pr-label.yml](.github/workflows/on-pr-label.yml): Triggers updates to PRs
  on specific labels in app repos ([internal usage
  docs](https://paotvers.io/docs/default/Domain/application-platform/Application/Repository/workflows/on-pr-label/))
- [misc-publish-dev-docker.yml](.github/workflows/misc-publish-dev-docker.yml):
  Publishes development Docker images to ACR ([internal usage
  docs](https://paotvers.io/docs/default/Domain/application-platform/Application/Repository/workflows/deployment-image-dev/))

### Other utility workflows usage docs

- [Deployment (dev image)](./docs/deployment-dev-image.md)
- [On PR label](./docs/on-pr-label.md)

## Archived workflows

The deprecated workflows were moved to
[.github/workflows/archived-workflows/](.github/workflows/archived-workflows/)
on 1 October 2026. They are not maintained or supported by the Platform team,
and contain known vulnerabilities.

GitHub does not resolve reusable workflows in subdirectories of
`.github/workflows`, so the archived files can no longer be called.
Callers must migrate to the golden path workflows:

| Archived | Migrate to |
| -------- | ---------- |
| `ci-pr-checks.yml` | [ci-pr-checks-lib.yml](.github/workflows/ci-pr-checks-lib.yml) (Maven libraries) and/or [ci-pr-checks-image.yml](.github/workflows/ci-pr-checks-image.yml) (containerized applications) |
| `ci-maven-build.yml`, `ci-maven-build-lib.yml` | [ci-pr-checks-lib.yml](.github/workflows/ci-pr-checks-lib.yml) |
| `ci-maven-deploy.yml`, `ci-maven-install-deploy-lib.yml` | [ci-build-publish-lib.yml](.github/workflows/ci-build-publish-lib.yml) |

`test-k6-build-docker.yml` and `test-k6-build-publish-docker.yml` were deleted.

## Deprecated custom workflows

- [ci-docker-build-publish-integrasjonspunkt.yml](.github/workflows/ci-docker-build-publish-integrasjonspunkt.yml)
- [ci-docker-build-scan-integrasjonspunkt.yml](.github/workflows/ci-docker-build-scan-integrasjonspunkt.yml)

Still used by an older integrasjonspunkt version, so they stay in
`.github/workflows` until eFormidling has moved its customers off it. They are
not maintained or supported by the Platform team. New callers should use the
`docker` type
[ci-build-publish-image.yml](.github/workflows/ci-build-publish-image.yml).

## Development guidelines

Check out the [internal usage
docs](https://paotvers.io/docs/default/Domain/application-platform/Github/Actions/development-guidelines/)
for more information.
