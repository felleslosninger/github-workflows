# GitHub Action: Image metadata

Author: **Digdir Platform Team**

## Description

This composite action generates Docker image metadata for workflows that need a consistent image name and tag.

It supports:

- Custom image tags via `image-tag`
- Explicit version strings via `version`
- Auto-generated tags (date + SHA) when no explicit tag is provided
- Container registry selection via `container-registry`
- Automatic `image-name` fallback to the current repository name

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `image-name` | Docker image name without registry. Defaults to repository name if unset. | false | `""` |
| `container-registry` | Container registry host (e.g. `creiddev.azurecr.io`, `ghcr.io`). | `true` | |
| `image-tag` | Custom image tag. Overrides auto-generation. | false | `""` |
| `version` | Use explicit version string as image tag when provided. | false | `""` |
| `auto-generate-tag` | Generate a tag from the date and SHA when no explicit tag is provided. | false | `true` |

## Outputs

| Output | Description |
| :----- | :---------- |
| `image-name` | Fully qualified image name including registry. |
| `image-tag` | Image tag. |

## Example usage

```yaml
steps:
  - name: Set image metadata
    id: image-metadata
    uses: felleslosninger/github-workflows/.github/actions/image-metadata@main
    with:
      image-name: my-app
      container-registry: creiddev.azurecr.io
      version: "1.2.0" # Optional: Will fallback to auto-generated tag if omitted
```

## How it works

The action generates the full image name (with registry) as `image-name`,
chooses the best available tag source as `image-tag`, and writes both values to
outputs for later build, scan, and publishing steps.
