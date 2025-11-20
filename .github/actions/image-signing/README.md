# GitHub Action: Keyless image signing using Cosign

Author: **Digdir Platform Team**

## Description

This GitHub Action is designed to sign our container images using keyless
signing with Cosign, Fulcio CA and Rekor transparency log. The signature and
attestation is then pushed to the container registry. The images can then be
audited with a tool like Kyverno to verify that the container image that is
deployed is actually built the way we expect.

## Prerequisites

- It is expected that this composite action is pulled into a workflow that has
  built a container image, as `docker inspect` needs the image to exist locally
- It is expected that this composite action is pulled into a workflow that has
  pushed a container image to a registry, as the workflow needs valid
  credentials to push the signature and attestation to the registry
- The image input string is expected to include the full image name in
  combination with a tag or digest (both also works)

Example inputs that should work

- `creiddev.azurecr.io/plattform-test-app:2025-11-20-1346-e2b80979@sha256:be7fc6cb642873797e783d12df606a2ed94c95b3eb4a6abe3912dbef2e4c224a`
- `creiddev.azurecr.io/plattform-test-app:2025-11-20-1346-e2b80979`
- `creiddev.azurecr.io/plattform-test-app@sha256:be7fc6cb642873797e783d12df606a2ed94c95b3eb4a6abe3912dbef2e4c224a`
- `creiddev.azurecr.io/plattform-test-app:v.1.0.0`

## Inputs

| Input | Description | Required |
| :--- | :--- | :--- |
| `image` | Valid image string for signing | true |

## Example Usage (external)

If you want to use this directly, you can do

```yaml
name: Some workflow that build and push container images

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      <other-steps-including-image-build-and-push>
      - name: Image signing
        uses: felleslosninger/github-workflows/actions/image-signing@main
        with:
          image: <image_string>
      <other-steps>
```

## Example Usage (internal)

If you want to use this within this repository, you can do

```yaml
name: Some workflow in this repository that build and push container images

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      <other-steps-including-image-build-and-push>
      - name: Image signing
        uses: ./.github/actions/image-signing
        with:
          image: <image_string>
      <other-steps>
```

## How it Works

This action utilizes a composite run to install Cosign, get metadata from the
image input using `docker inspect` and other tools, generate provenance metadata
and sign the image.
