# GitHub Action: Azure Container Registry Login

Author: **Digdir Platform Team**

## Description

This GitHub Action is designed to login to Azure Container Registry via federated credentials

## Prerequisites

- The job including the composite action must have the necessary `id-token`
  permission.

  ```yaml
  jobs:
    <job-name>:
      permissions:
        id-token: write
  ```

- The workflow must have access to the Azure credentials used for the
  `client-id`, `tenant-id` and `subscription-id` inputs. These are
  stored as organization `secrets` and referenced in the
  workflow as `${{ secrets.AZURE_CLIENT_ID }}`, `${{ secrets.AZURE_TENANT_ID }}`
  and `${{ secrets.AZURE_SUBSCRIPTION_ID }}`.

## Inputs

The `action.yml` inputs are:

| Input | Description | Required | Default |
| :--- | :--- | :---: | :--- |
| `client-id` | Azure Client ID | true | |
| `tenant-id` | Azure Tenant ID | true | |
| `subscription-id` | Azure Subscription ID | true | |
| `acr-name` | Azure container registry name | false | `creiddev` |

## Example Usage

```yaml
steps:
  - name: ACR login
    uses: felleslosninger/github-workflows/.github/actions/acr-login@main
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

## How it Works

This action performs two main steps using a composite action defined in
`action.yml`:

- **Login to Azure**: The action calls `azure/login` and passes the
  `client-id`, `tenant-id`, and `subscription-id` inputs as environment
  variables. This authenticates the workflow to the target Azure
  subscription using the provided service principal credentials.

- **Login to ACR**: After authentication, the action runs `az acr login`
  against the registry name provided by the `acr-name` input (defaults to
  `creiddev`). This ensures subsequent steps in the job can push and pull
  container images from the registry.


