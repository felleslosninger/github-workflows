# GitHub Action: Push Docker image

Author: **Digdir Platform Team**

## Description

This composite action pushes a Docker image to a container registry and extracts
its digest (`RepoDigest`) for downstream consumption (such as signing,
attestations, or deployments).

It is designed to work seamlessly with the outputs from the `image-metadata` action.

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `image-name` | Fully qualified image name including registry | `true` | |
| `image-tag` | The image tag to push | `true` | |

## Outputs

| Output | Description |
| :----- | :---------- |
| `image-digest` | The SHA256 digest of the pushed image |

## Example usage

```yaml
steps:
  - name: Set image metadata
    id: image-metadata
    uses: felleslosninger/github-workflows/.github/actions/image-metadata@main
    with:
      image-name: my-app
      container-registry: ghcr.io

  # (Build step goes here)

  - name: Push image
    id: push-image
    uses: felleslosninger/github-workflows/.github/actions/push-image@main
    with:
      image-name: ${{ steps.image-metadata.outputs.image-name }}
      image-tag: ${{ steps.image-metadata.outputs.image-tag }}

  - name: Use digest
    run: echo "The pushed image digest is ${{ steps.push-image.outputs.image-digest }}"
```

## How it works

The action performs a `docker push` using the provided fully-qualified image
name and tag. Once pushed, it uses `docker inspect` to parse the `RepoDigests`
field, extracting the specific SHA256 digest string and exporting it as an
output for subsequent steps.
