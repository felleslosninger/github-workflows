# Deployment usage docs (image)

These docs explain how to use our reusable workflow for container image
deployment in app repos

- [Build and publish container images](../.github/workflows/ci-build-publish-image.yml)
- [CD repo update version](../.github/workflows/ci-call-update-image.yml)

These workflows are split into several jobs.

Build and publish container images:

- Write inputs to step summary (no overrides necessary)
- Input validation (no overrides necessary)
- Build and publish container image (based on application type)

CD repo update version:

- Write inputs to step summary
- Send update image event to CD repo

## Usage

In your application repository, create a new file called
`.github/workflows/deploy.yml` (best practice naming).

### Single CD repo update

You normally build and push an image to a container registry and update
Kubernetes manifests for a single app in a single CD repo

```yaml
name: Deploy image

on:
  push:
    branches:
      - main

jobs:
  build-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-build-publish-image.yml@main
    permissions:
      contents: write
      packages: write
      id-token: write
    secrets: inherit

  update-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-call-update-image.yml@main
    permissions:
      pull-requests: read
    needs: build-image
    with:
      application-name: app-a # Application name (e.g. plattform-test-app)
      product-name: product-a # Product name (e.g. plattform)
      image-name: app-a # Image name is normally the same as the application name (e.g. plattform-test-app)
      image-version: ${{ needs.build-image.outputs.image-version }}
      image-digest: ${{ needs.build-image.outputs.image-digest }}
      kubernetes-repo: product-a-cd # CD repo name (e.g. plattform-cd)
      deployment-environment: systest # First deployment env (e.g. systest/dev or test)
    secrets: inherit
```

### Multiple CD repo updates

In some special cases you build and push an image that is deployed as separate
apps from several CD repos. In these cases, the CD repo update is handled with
the matrix strategy

```yaml
name: Deploy image

on:
  push:
    branches:
      - main

jobs:
  build-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-build-publish-image.yml@main
    permissions:
      contents: write
      packages: write
      id-token: write
    secrets: inherit

  update-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-call-update-image.yml@main
    permissions:
      pull-requests: read
    needs: build-image
    strategy:
      matrix:
        application-name: [app-a, app-b, app-c]
        include:
          - application-name: app-a
            kubernetes-repo: product-a-cd
            product-name: product-a
            deployment-environment: systest
          - application-name: app-b
            kubernetes-repo: product-b-cd
            product-name: product-b
            deployment-environment: systest
          - application-name: app-c
            kubernetes-repo: product-c-cd
            product-name: product-c
            deployment-environment: dev
    with:
      application-name: ${{ matrix.application-name }}
      product-name: ${{ matrix.product-name }}
      image-name: ${{ matrix.application-name }}
      image-version: ${{ needs.build-image.outputs.image-version }}
      image-digest: ${{ needs.build-image.outputs.image-digest }}
      kubernetes-repo: ${{ matrix.kubernetes-repo }}
      deployment-environment: ${{ matrix.deployment-environment }}
    secrets: inherit
```

## Overrides

Overrides are only supported for the build/publish workflow jobs.

### Build and publish container images

This is handled via 3 separate workflow jobs depending on application type

- `run-spring-boot-build`
- `run-quarkus-build`
- `run-docker-build`

#### Build and publish container images (common overrides)

##### Application type

| Override | Workflow default |
| -------- | ---------------- |
| `application-type` | `spring-boot` |

> [!NOTE]
> Valid values are `spring-boot`, `quarkus` or `docker`.

##### Container registry

| Override | Workflow default |
| -------- | ---------------- |
| `cr-type` | `acr` |
| `lifecycle` | `deployment` |

> [!TIP]
> By overriding `lifecycle` to `development` you can deploy branch images to the
> `systest` cluster for testing purposes. Remember to override `lifecycle` to
> `development` in the `update-image` workflow job as well. See the examples
> section on how to do this.

##### Image metadata

| Override | Workflow default |
| -------- | ---------------- |
| `image-name` | `` |
| `image-tag` | `` |

> [!NOTE]
> Leaving `image-name` empty will default this to the GitHub repository name.
> It is only necessary to override this if you want a different name for your
> image. Leaving `image-tag` empty will default this to a generated best
> practice tag.

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

##### Image signing

| Override | Workflow default |
| -------- | ---------------- |
| `image-signing` | `true` |

> [!NOTE]
> Image signing should normally never be disabled as this is a requirement for
> deploying to production environments.

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

##### Slack channel ID

| Override | Workflow default |
| -------- | ---------------- |
| `slack-channel-id` | `` |

> [!TIP]
> Override this to your team CI/CD Slack channel to get Slack notifications on
> failed build/publish workflow runs. You can use either name or ID. In addition
> to the team channel, all build/publish workflow run failures will also be sent
> to the Platform team CI/CD channel.

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

#### Build and publish container images (Spring Boot overrides)

| Override | Workflow default |
| -------- | ---------------- |
| `update-versions` | `true` |
| `module-name` | `` |

#### Build and publish container images (Quarkus overrides)

| Override | Workflow default |
| -------- | ---------------- |
| `native` | `false` |

#### Build and publish container images (Docker overrides)

| Override | Workflow default |
| -------- | ---------------- |
| `add-git-package-token` | `false` |
