# GitHub Action: Image metadata

Author: **Digdir Platform Team**

## Description

This composite action generates Docker image metadata for workflows that need a
consistent image name and tag.

It supports

- Custom image tags via `image-tag`
- Package-version tags via `package-version`
- Explicit version strings via `version`
- Snapshot stripping when building from `main` or tag refs
- Auto-generated tags when no explicit tag is provided
- Container registry selection via `container-registry` or `registry-url`
- Automatic `image-name` fallback to the current repository name

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `image-name` | Docker image name without registry. Defaults to repository name if unset. | false | `""` |
| `container-registry` | Container registry host (e.g. `creiddev.azurecr.io`, `ghcr.io`). | false | `""` |
| `registry-url` | Alternate registry URL if `container-registry` is not provided. | false | `""` |
| `image-tag` | Custom image tag. Overrides auto-generation. | false | `""` |
| `package-version` | Use package version as image tag when provided. | false | `""` |
| `version` | Use explicit version string as image tag when provided. | false | `""` |
| `version-pom-path` | Evaluate Maven `pom.xml` to derive the version when no explicit tag is provided. | false | `` |
| `strip-snapshot` | Strip `-SNAPSHOT` from version when building from `main` or tag refs. | false | `false` |
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
```

## How it works

The action validates registry and image-name inputs, chooses the best available
tag source, and writes both values to outputs for later build, scan, and
publishing steps.
