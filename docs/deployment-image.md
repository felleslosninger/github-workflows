# Deployment usage docs (image)

These docs explain how to use our reusable workflow for container image
deployment in app repos.

- [Build and publish container images](../.github/workflows/ci-build-publish-image.yml)
- [CD repo update version](../.github/workflows/ci-call-update-image.yml)

These workflows are split into several jobs.

Build and publish container images:

- Write inputs to step summary (no overrides supported)
- Input validation (no overrides supported)
- Build and publish container image (based on application type)

CD repo update version:

- Write inputs to step summary (no overrides supported)
- Send update image event to CD repo

## GitHub workflow permissions

This is the list of the current workflow permissions used as part of these
workflows.

| Permission | Purpose |
| ---------- | ------- |
| `contents: write` | Needed for uploading SBOM to GitHub's Dependency Graph |
| `packages: write` | Needed to allow container image pushes to GitHub container registry  (currently supported for `spring-boot` apps) |
| `id-token: write` | Needed when using OIDC (federated credentials) to log in to ACR and for image signing with Sigstore/Cosign |
| `pull-requests: read` | Needed to fetch PR metadata used in the deployment payload |

## GitHub secrets

This is the list of the current GitHub secrets used as part of these workflows.

| Secret | Type | Visibility | Admin action required |
| ------ | ---- | ---------- | --------------------- |
| `DIGDIR_PLATFORM_CI_APP_ID` | Org. secret | All repositories | No |
| `DIGDIR_PLATFORM_CI_APP_PRIVATE_KEY` | Org. secret | Per repository | Yes |
| `GH_PACKAGES_READ_USER` | Org. secret | All repositories | No |
| `GH_PACKAGES_READ_PAT` | Org. secret | All repositories | No |
| `AZURE_TENANT_ID` | Org. secret | All repositories | No |
| `AZURE_SUBSCRIPTION_ID` | Org. secret | All repositories | No |
| `AZURE_CLIENT_ID` | Org. secret | All repositories | No |
| `SLACK_CICD_NOTIFICATION_TOKEN` | Org. secret | All repositories | No |

> [!WARNING]
> A GitHub admin must add the `DIGDIR_PLATFORM_CI_APP_PRIVATE_KEY`
> repository-scoped organization secret to new application repositories that
> want to use these workflows.

## Usage

In your application repository, create a new file called
`.github/workflows/deploy.yml` (best practice naming).

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

> [!NOTE]
>
> **Supported repository type:** Because this configuration uses all the
> workflow defaults, it assumes the repository is a **single-module Spring Boot
> application** located at the repository root. It expects a standard Maven
> setup with a `pom.xml` in the root directory, built using the default Java
> version. If your repository is a multi-module project, a monorepo, a
> Quarkus/Docker application, need to update multiple CD repos, or uses a
> different Java version, you must provide the necessary overrides. See the
> [**Examples**](#examples) section below on how to configure these specific
> setups.

## Overrides

Overrides are only supported for the build/publish and update version workflow
jobs.

### Build and publish container images

This is handled via 3 separate workflow jobs depending on application type:

- `run-spring-boot-build`
- `run-quarkus-build`
- `run-docker-build`

One of these will always run. The 3 jobs support some common overrides, as well
as overrides based on the application type.

#### Build and publish container images (common overrides)

##### Application type

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `application-type` | `string` | `spring-boot` |

> [!NOTE]
> Valid values are `spring-boot`, `quarkus` or `docker`.

##### Container registry

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `cr-type` | `string` | `acr` |
| `lifecycle` | `string` | `deployment` |

> [!TIP]
> By overriding `lifecycle` to `development` you can deploy branch images to the
> `systest` cluster for testing purposes. Remember to override `lifecycle` to
> `development` in the `update-image` workflow job as well. See the examples
> section on how to do this.

##### Image metadata (build/publish)

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `image-name` | `string` | Not set |
| `image-tag` | `string` | Not set |

> [!NOTE]
> Leaving `image-name` empty will default this to the GitHub repository name. It
> is only necessary to override this if you want a different name for your
> image. Leaving `image-tag` empty will default this to a generated best
> practice tag.

##### Application path

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `application-path` | `string` | `./` |

> [!CAUTION]
> **Path formatting:** The path is relative to the repository root. If
> overridden for Spring Boot/Quarkus, you **must include a trailing slash**
> (e.g., `backend/`). The underlying scripts concatenate this directly
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

##### Image signing

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `image-signing` | `boolean` | `true` |

> [!NOTE]
> Image signing should normally never be disabled as this is a requirement for
> deploying to production environments.

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
> current default. The cache path should be overridden for monorepo workflows
> where an app is self-contained within a repo directory with a separate
> `pom.xml`.

##### Slack channel ID

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `slack-channel-id` | `string` | Not set |

> [!TIP]
> Override this to your team CI/CD Slack channel to get Slack notifications on
> failed build/publish workflow runs. You can use either name or ID. In addition
> to the team channel, all build/publish workflow run failures will also be sent
> to the Platform team CI/CD channel.

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

#### Build and publish container images (Spring Boot overrides)

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `update-versions` | `string` | `true` |
| `module-name` | `string` | Not set |

> [!CAUTION]
>
> - **Update versions:** If `update-versions` is `true`, the workflow runs `mvn
>   versions:set -DnewVersion="$IMAGE_TAG"`. This modifies your `pom.xml`
>   dynamically during the build to inject the generated container tag. Ensure
>   your POM structure supports this plugin safely.
>
> - **Multi-module projects:** Use `module-name` instead of `application-path`
>   if your app is part of a Maven multi-module project. This instructs the
>   workflow to use the `-pl <module> -am` flags, ensuring internal dependencies
>   are built correctly before the image is packaged.

#### Build and publish container images (Quarkus overrides)

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `native` | `boolean` | `false` |

> [!NOTE]
> Setting `native` to `true` compiles a GraalVM native image using Paketo
> buildpacks. Be aware that native compilation is highly resource-intensive and
> will significantly increase build times.

#### Build and publish container images (Docker overrides)

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `add-git-package-token` | `boolean` | `false` |
| `docker-build-context` | `string` | `.` |
| `sh-scan-install-method` | `string` | `npm` |
| `sh-scan` | `boolean` | `true` |

> [!CAUTION]
> **Dockerfile path resolution:** If you set `add-git-package-token` to `true`,
> the workflow *strictly* looks for the Dockerfile at `docker/Dockerfile`
> relative to the build context. Ensure your repository matches this structure.

<!-- -->

> [!NOTE]
> `docker-build-context` sets the directory `docker build` uses to resolve
> `COPY`/`ADD` paths — separate from `application-path`, which only sets where
> the Dockerfile itself is. Override it if your Dockerfile copies files from
> outside the repository root.
> If you are building a node project with a different package manager than npm,
> ex. `yarn`, you have to set `sh-scan-install-method` to `yarn`. It is also
> possible to skip the sh-scan-step by setting `sh-scan` to `false`.

### CD repo update version

This is handled via a separate workflow job called `prepare-payload`. This will
always run.

#### CD repo update version overrides

##### Application name

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `application-name` | `string` | Not set |

> [!CAUTION]
> This must match the application folder name in the CD repo, or the update
> version pipeline will fail for the image update.

##### Product name

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `product-name` | `string` | Not set |

> [!CAUTION]
> This must match the product folder name in the CD repo, or the update version
> pipeline will fail for the image update.

##### Image metadata (update version)

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `image-name` | `string` | Not set |
| `image-version` | `string` | Not set |
| `image-digest` | `string` | Not set |

> [!CAUTION]
> Image name must match what was used when the image was built in the
> build/publish workflow job, or the wrong image name will be used when ArgoCD
> tries to deploy the new version. The build/publish workflow job generates
> outputs for image version and digest which MUST be mapped into these inputs
> (e.g., `image-version: ${{ needs.build-image.outputs.image-version }}`).

##### Kubernetes repo

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `kubernetes-repo` | `string` | Not set |
| `kubernetes-repo-event` | `string` | `update-version` |

> [!CAUTION]
> Kubernetes repo must match the CD repo that is responsible for deploying the
> application that is being built. The Kubernetes repo event should only be
> overridden for testing purposes.

##### Deployment environment

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `deployment-environment` | `string` | Not set |

> [!CAUTION]
> This must match the environment folder name in the CD repo, or the update
> version pipeline will fail for the image update.

##### Kustomize version

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `kustomize-version` | `string` | Not set |

> [!NOTE]
> The Kustomize version should only be overridden for testing purposes (it
> defaults to following the ArgoCD version).

##### Lifecycle

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `lifecycle` | `string` | `deployment` |

> [!CAUTION]
> This must match the lifecycle used for the build/publish workflow job, or the
> wrong container registry will be passed into the CD payload.

## Examples

### Multiple CD repo updates

In some special cases you build and push an image that is deployed as separate
apps from several CD repos. In these cases, the CD repo update is handled with
the matrix strategy:

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

### Deploying images from a branch

By using the `lifecycle` override on both workflow jobs, you can push container
images to our development container registry. This registry is allowed as a
source of deployments in our `systest` cluster. If you want to utilize this
functionality, you can create a separate workflow for this, in addition to the
one you use for production images.

Create a new file called `.github/workflows/deploy-dev.yml` (best practice
naming):

```yaml
name: Deploy image (dev)

on:
  workflow_dispatch:

jobs:
  build-image-dev:
    uses: felleslosninger/github-workflows/.github/workflows/ci-build-publish-image.yml@main
    permissions:
      contents: write
      packages: write
      id-token: write
    with:
      lifecycle: development
    secrets: inherit

  update-image-dev:
    uses: felleslosninger/github-workflows/.github/workflows/ci-call-update-image.yml@main
    permissions:
      pull-requests: read
    needs: build-image-dev
    with:
      application-name: app-a # Application name (e.g. plattform-test-app)
      product-name: product-a # Product name (e.g. plattform)
      image-name: app-a # Image name is normally the same as the application name (e.g. plattform-test-app)
      image-version: ${{ needs.build-image-dev.outputs.image-version }}
      image-digest: ${{ needs.build-image-dev.outputs.image-digest }}
      kubernetes-repo: product-a-cd # CD repo name (e.g. plattform-cd)
      deployment-environment: systest # First deployment env (e.g. systest/dev or test)
      lifecycle: development
    secrets: inherit
```

> [!NOTE]
> By using a workflow dispatch trigger, you can manually run this when needed,
> and choose the branch that you want to produce a container image from. The
> image will be pushed to our development container registry, and the CD repo
> will be updated in the same way as with a production container image. When
> lifecycle is overridden to `development`, no promotion PRs will be created in
> your CD repo.

### Multi-module Maven project

Use this setup when your repository has a root `pom.xml` and multiple
interconnected modules, but you are only building a container image for one
specific module (e.g., `api`).

This uses `module-name` to build internal dependencies first.

```yaml
name: Deploy API image

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
    with:
      # Triggers 'mvn ... -pl "api" -am' to build internal dependencies first
      module-name: api
    secrets: inherit

  update-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-call-update-image.yml@main
    permissions:
      pull-requests: read
    needs: build-image
    with:
      application-name: api
      product-name: my-product
      image-name: api
      image-version: ${{ needs.build-image.outputs.image-version }}
      image-digest: ${{ needs.build-image.outputs.image-digest }}
      kubernetes-repo: my-product-cd
      deployment-environment: systest
    secrets: inherit
```

### Monorepo with independent applications

Use this setup when your repository contains completely separate applications
that do *not* share a root `pom.xml` or internal dependencies.

For monorepos, you should create a separate workflow file for each application
and use the `paths` trigger. This ensures that changes to `Application A` do not
unnecessarily trigger the build, Trivy scans, and consume GitHub Actions minutes
for `Application B`.

**`.github/workflows/deploy-backend-service.yml`**

```yaml
name: Deploy Backend Service

on:
  push:
    branches:
      - main
    paths:
      - "apps/backend-service/**"
      - ".github/workflows/deploy-backend-service.yml"

jobs:
  build-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-build-publish-image.yml@main
    permissions:
      contents: write
      packages: write
      id-token: write
    with:
      application-path: "apps/backend-service/"
      cache-path: "apps/backend-service/pom.xml"
    secrets: inherit

  update-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-call-update-image.yml@main
    permissions:
      pull-requests: read
    needs: build-image
    with:
      application-name: backend-service
      product-name: my-product
      image-name: backend-service
      image-version: ${{ needs.build-image.outputs.image-version }}
      image-digest: ${{ needs.build-image.outputs.image-digest }}
      kubernetes-repo: my-product-cd
      deployment-environment: systest
    secrets: inherit
```

**`.github/workflows/deploy-auth-service.yml`**

```yaml
name: Deploy Auth Service

on:
  push:
    branches:
      - main
    paths:
      - "apps/auth-service/**"
      - ".github/workflows/deploy-auth-service.yml"

jobs:
  build-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-build-publish-image.yml@main
    permissions:
      contents: write
      packages: write
      id-token: write
    with:
      application-path: "apps/auth-service/"
      cache-path: "apps/auth-service/pom.xml"
    secrets: inherit

  update-image:
    uses: felleslosninger/github-workflows/.github/workflows/ci-call-update-image.yml@main
    permissions:
      pull-requests: read
    needs: build-image
    with:
      application-name: auth-service
      product-name: my-product
      image-name: auth-service
      image-version: ${{ needs.build-image.outputs.image-version }}
      image-digest: ${{ needs.build-image.outputs.image-digest }}
      kubernetes-repo: my-product-cd
      deployment-environment: systest
    secrets: inherit
```

> [!CAUTION]
> **Path formatting:** When overriding `application-path`, you **must** include
> the trailing slash (e.g., `apps/backend-service/`). The underlying scripts
> concatenate this directly (evaluating to `${APP_PATH}pom.xml`), and the build
> will fail if the slash is omitted.
