# Deployment usage docs (dev image)

These docs explain how to use our reusable workflow for development container
image deployment in app repos

- [Build and publish development container image](../.github/workflows/misc-publish-dev-docker.yml)

This workflow runs on main commits in an application repository, and will
publish images to our `crutvikling.azurecr.io` development Azure Container
Registry (ACR). The purpose of this workflow is to push a latest image to a
place where developers have access, so they can run it locally.

## Usage

In your application repository, create a new file called
`.github/workflows/deploy-dev.yml` (best practice naming)

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
    uses: felleslosninger/github-workflows/.github/workflows/misc-publish-dev-docker.yml@main
    secrets: inherit
```

## Overrides

### Build and publish development container images

This is handled via a separate workflow job called `docker-build-publish`. This
will always run.

#### Build and publish development container images overrides

##### Dockerfile path

| Override | Workflow default |
| -------- | ---------------- |
| `docker-file` | `docker/dev.Dockerfile` |
