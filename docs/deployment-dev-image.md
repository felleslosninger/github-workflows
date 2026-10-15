# Deployment usage docs (dev image)

These docs explain how to use our reusable workflow for building development
container images mainly used for local development purposes.

- [Build and publish development container image](../.github/workflows/misc-publish-dev-docker.yml)

This workflow runs on main commits in an application repository, and will
publish multi-architecture images to our development Azure Container Registry
(ACR). The purpose of this workflow is to push a `latest` image to a registry
where developers have access, allowing them to easily run and test it locally.

## GitHub workflow permissions

This is the list of the current workflow permissions used as part of this
workflow.

| Permission | Purpose |
| ---------- | ------- |
| `contents: read` | Needed to check out the repository |
| `id-token: write` | Needed when using OIDC (federated credentials) to log in to ACR |

> [!IMPORTANT]
> The calling job must grant these permissions. If it grants fewer, GitHub
> rejects the whole calling workflow file, not just this job.

## GitHub secrets

This is the list of the current GitHub secrets used as part of this workflow.

| Secret | Type | Visibility | Admin action required |
| ------ | ---- | ---------- | --------------------- |
| `CR_DEV_USERNAME` | Org. secret | All repositories | No |
| `AZURE_TENANT_ID` | Org. secret | All repositories | No |
| `AZURE_SUBSCRIPTION_ID` | Org. secret | All repositories | No |
| `GH_PACKAGES_READ_USER` | Org. secret | All repositories | No |
| `GH_PACKAGES_READ_PAT` | Org. secret | All repositories | No |

> [!NOTE]
> No GitHub admin action is required to use this workflow.

## Usage

In your application repository, create a new file called
`.github/workflows/deploy-dev.yml` (best practice naming).

```yaml
name: Publish development Docker image

on:
  push:
    branches:
      - "main"
    paths-ignore:
      - 'src/test/**'
      - '*.md'
      - 'LICENSE'
      - "catalog-info.yaml"
      # Add more as needed

jobs:
  build-publish-dev-image:
    permissions:
      contents: read
      id-token: write
    uses: felleslosninger/github-workflows/.github/workflows/misc-publish-dev-docker.yml@main
    secrets: inherit
```

> [!NOTE]
>
> - **Multi-platform support:** The underlying workflow automatically sets up
>   QEMU and Docker Buildx to build and push images for both `linux/amd64` and
>   `linux/arm64` architectures.
>
> - **Tagging:** Images built with this workflow are always tagged as `latest`
>   in the development ACR.

## Overrides

### Build and publish development container images

This is handled via a separate workflow job called `docker-build-publish`. This
will always run.

#### Build and publish development container images overrides

##### Dockerfile path

| Override | Type | Workflow default |
| -------- | ---- | ---------------- |
| `docker-file` | `string` | `docker/dev.Dockerfile` |

> [!CAUTION]
> If you override `docker-file`, ensure the path points correctly to your custom
> Dockerfile relative to the repository root.
