# github-workflows

## Overview

This repository contains reusable GitHub Workflows and composite Actions for
common CI/CD tasks. Workflows are organized by purpose and application type.

## Composite actions

We have a set of composite actions that contain reusable steps to avoid
duplication, and to reduce maintenance for the Platform team

- [acr-login](./.github/actions/acr-login/): This composite action is designed
  to login to Azure Container Registry via federated credentials
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
- [zap-scan](./.github/actions/zap-scan): This composite action is designed to
  perform ZAP (OWASP Zed Attack Proxy) Dynamic Application Security Testing
  (DAST) scanning

These actions are used as part of our golden path workflows, but can also be
included in custom workflows depending on your needs. Check out the READMEs in
the corresponding action directories for more information on how to use them.

## Golden path workflows (recommended)

For new projects, you should use the following workflows depending on your needs.

### PR Checks

[ci-pr-checks.yml](.github/workflows/ci-pr-checks.yml): Golden path for PRs to
main. Handles Maven library builds and containerized applications. Verifies PR
titles, runs builds and security scans, optionally builds container images with
Paketo buildpacks, and can auto-merge Dependabot PRs. This workflow will trigger
one of the following application-specific workflows

- [ci-spring-boot-container-scan.yml](.github/workflows/ci-spring-boot-container-scan.yml):
  Builds and scans temporary Spring Boot container images
- [ci-quarkus-container-scan.yml](.github/workflows/ci-quarkus-container-scan.yml):
  Builds and scans temporary Quarkus JVM or GraalVM native container images

and optionally runs the Dependabot auto-merge workflow

- [misc-approve-and-merge-dependabot-pr.yml](.github/workflows/misc-approve-and-merge-dependabot-pr.yml):
  Auto-approves and merges supported Dependabot updates

Check out the [internal usage
docs](https://paotvers.io/docs/default/Domain/application-platform/Application/Repository/workflows/pull-request-image/)
for more information.

### Build and publish container images

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

Check out the [internal usage
docs](https://paotvers.io/docs/default/Domain/application-platform/Application/Repository/workflows/deployment-image/)
for more information.

### Build and publish Maven libraries

Workflows for building and publishing Maven libraries after commits to main

- [ci-maven-deploy.yml](.github/workflows/ci-maven-deploy.yml): For projects
  without internal Maven dependencies
- [ci-maven-install-deploy-lib.yml](.github/workflows/ci-maven-install-deploy-lib.yml):
  For projects with internal dependencies

Check out the [internal usage
docs](https://paotvers.io/docs/default/Domain/application-platform/Application/Repository/workflows/release-artifact/)
for more information.

## Other utility workflows

- [check-syntax.yml](.github/workflows/check-syntax.yml): Validates workflow
  files with actionlint in this repository (local workflow)
- [on-pr-label.yml](.github/workflows/on-pr-label.yml): Marks internal PRs when
  labeled ([internal usage docs](https://paotvers.io/docs/default/Domain/application-platform/Application/Repository/workflows/on-pr-label/))
- [misc-publish-dev-docker.yml](.github/workflows/misc-publish-dev-docker.yml):
  Publishes development Docker images to ACR ([internal usage docs](https://paotvers.io/docs/default/Domain/application-platform/Application/Repository/workflows/deployment-image-dev/))

## Deprecated Workflows

### Maven workflows

The following Maven PR workflows can be considered deprecated as all
functionality should be covered by our golde path
[ci-pr-checks.yml](.github/workflows/ci-pr-checks.yml) workflow

- [ci-maven-build.yml](.github/workflows/ci-maven-build.yml)
- [ci-maven-build-lib.yml](.github/workflows/ci-maven-build-lib.yml)

### Custom workflows

The following workflows are not really maintained by the Platform team, and app
repos using them should most likely migrate to the `docker` type
[ci-build-publish-image.yml](.github/workflows/ci-build-publish-image.yml)
workflow

- [ci-docker-build-publish-integrasjonspunkt.yml](.github/workflows/ci-docker-build-publish-integrasjonspunkt.yml)
- [ci-docker-build-scan-integrasjonspunkt](.github/workflows/ci-docker-build-scan-integrasjonspunkt)
- [test-k6-build-docker.yml](.github/workflows/test-k6-build-docker.yml)
- [test-k6-build-publish-docker.yml](.github/workflows/test-k6-build-publish-docker.yml)

Note that the Platform team might still update these to use new composite
actions when applicable.
