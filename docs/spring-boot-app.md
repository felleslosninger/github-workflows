# How to configure Spring boot java application

Create folder `.github/workflows/` folder in your application repo if not already created.

## Build branch when pull request created

Possible inputs for `call-workflow-maven-build`:
      java-version: "<value>" (OPTIONAL)
        default: '11'

Possible inputs for `call-container-scan`:
      image-name: "<value>" (OPTIONAL, inherits repository name if not set)
      image-pack: "<value>" (OPTIONAL)
        default: builder-jammy-tiny
      java-version: "<value>" (OPTIONAL)
        default: '11'

Add workflow file `.github/workflows/call-maventests.yml` with the following content:

```
name: Build Maven Java

on:
    pull_request:
        branches: [ main ]

jobs:
  call-workflow-maven-build:
    uses: felleslosninger/github-workflows/.github/workflows/ci-maven-build.yml@main
    # with: 
      # <inputs>
    secrets: inherit
  call-container-scan:
    uses: felleslosninger/github-workflows/.github/workflows/ci-spring-boot-container-scan.yml@main
    # with: 
      # <inputs>
    secrets: inherit
    
```
For more details see description in workflow [ci-maven-build.yml](../.github/workflows/ci-maven-build.yml) and [ci-spring-boot-container-scan.yml](../.github/workflows/ci-spring-boot-container-scan.yml).


## Build and push image and update kubernetes configuration with new image version

Possible inputs for `call-workflow-image-build-publish``: 
      image-name: "<value>" (OPTIONAL, inherits repository name if not set)
      image-pack: "<value>" (OPTIONAL)
        default: builder-jammy-tiny
      java-version: "<value>" (OPTIONAL)
        default: '11'
      slack-channel-id: "<value>" (OPTIONAL)

Possible inputs for `call-update-image-version``: 
      application-name: "<value>" (MANDATORY)
      product-name: "<value>" (MANDATORY)
      image-name: "<value>" (MANDATORY)
      kubernetes-repo: "<name of CD repo>" (MANDATORY)
      deployment-environment: "<value>" (MANDATORY)

Example of workflow file `.github/workflows/call-buildimage.yml` with all values set

```
name: Build/publish Docker image

on:
  push:
    branches: [ main ]

jobs:
  call-workflow-image-build-publish:
    uses: felleslosninger/github-workflows/.github/workflows/ci-spring-boot-build-publish-image.yml@main
    # with:
      # inputs
    secrets: inherit

  call-update-image-version:
    uses: felleslosninger/github-workflows/.github/workflows/ci-call-update-image.yml@main
    needs: [call-workflow-image-build-publish]
    with:
      # inputs
      image-version: ${{ needs.call-workflow-image-build-publish.outputs.image-version }}
      image-digest: ${{ needs.call-workflow-image-build-publish.outputs.image-digest }}
    secrets: inherit
```
For more details see description in workflow [ci-spring-boot-build-publish-image.yml](../.github/workflows/spring-boot-build-publish-image.yml) and [ci-call-update-image.yml](../.github/workflows/ci-call-update-image.yml).

