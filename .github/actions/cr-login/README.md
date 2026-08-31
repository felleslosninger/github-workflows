# GitHub Action: Container Registry Login

Author: **Digdir Platform Team**

## Description

This GitHub Action is designed to log into either an Azure Container Registry
(ACR) via federated credentials or the GitHub Container Registry (GHCR) using a
standard GitHub token. It routes the login automatically based on the provided
registry input.

## Prerequisites

- For **ACR** logins, the job must have the necessary `id-token` permission to
  use federated OIDC credentials.
- For **GHCR** logins, the job must have the appropriate `packages: write` or
  `packages: read` permission.

  ```yaml
  jobs:
    <job-name>:
      permissions:
        id-token: write
        packages: write
  ```

- For **ACR**, the workflow must have access to the Azure credentials used for
  the `client-id`, `tenant-id`, and `subscription-id` inputs. These are stored
  as organization secrets and referenced in the workflow as `${{
  secrets.AZURE_CLIENT_ID }}`, `${{ secrets.AZURE_TENANT_ID }}`, and `${{
  secrets.AZURE_SUBSCRIPTION_ID }}`.

## Inputs

The `action.yml` inputs are:

| Input | Description | Required | Default |
| :--- | :--- | :---: | :--- |
| `registry` | The container registry to log into (e.g., `ghcr.io` or `creiddev`) | true | |
| `client-id` | Azure Client ID (Required for ACR) | false | |
| `tenant-id` | Azure Tenant ID (Required for ACR) | false | |
| `subscription-id` | Azure Subscription ID (Required for ACR) | false | |
| `github-token` | GitHub Token used for GHCR authentication | false | `${{ github.token }}` |

## Example Usage

### Single-step dynamic routing (Recommended)

This approach handles both registries dynamically based on the input variable.
Even if `container-registry` resolves to `ghcr.io`, passing the Azure secrets is
safe because the composite action will ignore them.

  ```yaml
  steps:
    - name: Container Registry Login
      uses: felleslosninger/github-workflows/.github/actions/cr-login@main
      with:
        registry: ${{ inputs.container-registry }}
        client-id: ${{ secrets[inputs.sp-container-registry-client-id] }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
