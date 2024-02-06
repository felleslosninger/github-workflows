# github-workflows

## Spring boot application workflows
For building branch on pull request created: [maven-build.yml](.github/workflows/ci-maven-build.yml)

Building and push image and update *-cd repository. Starts when branch is merged to main: [ci-spring-boot-build-publish-image.yml](.github/workflows/ci-spring-boot-build-publish-image.yml) and [ci-spring-boot-container-scan.yml](.github/workflows/ci-spring-boot-container-scan.yml).

See [how to configure](docs/spring-boot-app.md) for more details and examples for Spring boot applications.

## Syntax check of the action in this repository
See [check-syntax.yml](.github/workflows/check-syntax.yml) for details. Only for internal use in current repository.
