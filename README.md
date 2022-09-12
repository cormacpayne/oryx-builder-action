# Oryx++ Builder GitHub Action

This action allows users to easily deploy their application source to an [Azure Container App](https://azure.microsoft.com/en-us/services/container-apps/) in their workflow without needing to manage a Dockerfile and/or container image.

The following steps are performed by this action:

- Logs in to Azure CLI and Azure Container Registry using the provided credentials
- Uses the Oryx++ Builder to build the application source using [Oryx](https://github.com/microsoft/Oryx) and produce a runnable image
- Pushes this runnable image to Azure Container Registry
- Creates or updates a Container App based on this image in Azure Container Registry

## Prerequisites

Due to the Azure authorization required for this task to successfully execute, a few [GitHub secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) will need to be set in the user's repository containing the GitHub Actions workflow.

For the [`azure/login`](https://github.com/marketplace/actions/azure-login) action used to authenticate calls to the Azure CLI, deployment credentials will need to be created and set as a GitHub secret. Depending on the arguments provided to the Oryx++ Builder GitHub Action, the credentials will need to have Contributor access over one of the following:

- The existing Container App if both the `resourceGroup` and `containerAppName` arguments are provided _and_ exist
- The existing resource group if only the `resourceGroup` argument is provided _and_ exists
- The subscription if the `resourceGroup` argument is provided _or_ is provided but doesn't exist 

More information about configuring the deployment credentials required for this GitHub Action can be found [here](https://github.com/marketplace/actions/azure-login#configure-deployment-credentials).

For the [`docker/login-action`](https://github.com/marketplace/actions/docker-login) action used to authenticate image push calls to Azure Container Registry, the name of the Azure Container Registry is required, along with username and password credentials that can be retrieved from [creating a service principal](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-service-principal#create-a-service-principal) with proper permissions to the ACR resource.

## Arguments

Below are the arguments that can be provided to the Oryx++ Builder GitHub Action:

| Argument name | Required | Description |
| ------------- | -------- | ----------- |
| `appSourcePath` | Yes | The path (on the current runner) to the source of the application to be built. |
| `azureCredentials` | Yes | Azure credentials used by the `azure/login` action to authenticate Azure CLI requests. |
| `acrName` | Yes | The name of the Azure Container Registry that the runnable application image will be pushed to. |
| `acrUsername` | Yes | The username used to authenticate push requests to the provided Azure Container Registry. |
| `acrPassword` | Yes | The password used to authenticate push requests to the provided Azure Container Registry. |
| `containerAppName` | No | The name of the Container App that will be created or updated. If not provided, this value will be `oryx-builder-<github-run-id>-<github-run-attempt>`. |
| `resourceGroup` | No | The resource group that the Container App will be created in, or currently exists in. If not provided, this value will be `<container-app-name>-rg`. |
| `containerAppEnvironment` | No | The name of the Container App environment to use with the application. If not provided, an existing environment in the resource group of the Container App will be used, otherwise, an environment will be created in the formation `<container-app-name>-env`. |
| `runtimeStack` | No | The platform version stack used in the final runnable application image that is deployed to the Container App. The value should be provided in the formation `<platform>:<version>`. If not provided, this value is determined by Oryx based on the contents of the provided application. Please refer to [this document](https://github.com/microsoft/Oryx/blob/main/doc/supportedRuntimeVersions.md) for more information on supported runtime stacks for Oryx. |
| `targetPort` | No | The target port that the Container App will listen on. If not provided, this value will be "80" for Python applications and "8080" for all other supported platforms. |

## Usage

See [`action.yml`](./action.yml)

### Minimal

```yml
steps:
  - name: Build and deploy Container App
    uses cormacpayne/oryx-builder-action@main
    with:
      appSourcePath: ${{ github.workspace }}
      azureCredentials: ${{ secrets.AZURE_CREDENTIALS }}
      acrName: mytestacr
      acrUsername: ${{ secrets.REGISTRY_USERNAME }}
      acrPassword: ${{ secrets.REGISTRY_PASSWORD }}
```

This will create a new Container App named `oryx-builder-<github-run-id>-<github-run-attempt>` in a new resource group named `<container-app-name>-rg`.

### Container App name provided

```yml
steps:
  - name: Build and deploy Container App
    uses cormacpayne/oryx-builder-action@main
    with:
      appSourcePath: ${{ github.workspace }}
      azureCredentials: ${{ secrets.AZURE_CREDENTIALS }}
      acrName: mytestacr
      acrUsername: ${{ secrets.REGISTRY_USERNAME }}
      acrPassword: ${{ secrets.REGISTRY_PASSWORD }}
      containerAppName: my-test-container-app
```

This will create a new Container App named `my-test-container-app` in a new resource group named `my-test-container-app-rg`.

### Resource group provided

```yml
steps:
  - name: Build and deploy Container App
    uses cormacpayne/oryx-builder-action@main
    with:
      appSourcePath: ${{ github.workspace }}
      azureCredentials: ${{ secrets.AZURE_CREDENTIALS }}
      acrName: mytestacr
      acrUsername: ${{ secrets.REGISTRY_USERNAME }}
      acrPassword: ${{ secrets.REGISTRY_PASSWORD }}
      resourceGroup: my-test-rg
```

This will create a new Container App named `oryx-builder-<github-run-id>-<github-run-attempt>` in a new resource group named `my-test-rg`.

### Container App name and resource group provided

```yml
steps:
  - name: Build and deploy Container App
    uses cormacpayne/oryx-builder-action@main
    with:
      appSourcePath: ${{ github.workspace }}
      azureCredentials: ${{ secrets.AZURE_CREDENTIALS }}
      acrName: mytestacr
      acrUsername: ${{ secrets.REGISTRY_USERNAME }}
      acrPassword: ${{ secrets.REGISTRY_PASSWORD }}
      containerAppName: my-test-container-app
      resourceGroup: my-test-rg
```

If the `my-test-rg` resource group does not exist, this will create the resource group and create a new Container App named `my-test-container-app` within the resource group. If the resource group already exists, this will create a new Container App named `my-test-container-app` in the resource group, or update the Container App if it already exists within the resource group.

### Container App environment provided

```yml
steps:
  - name: Build and deploy Container App
    uses cormacpayne/oryx-builder-action@main
    with:
      appSourcePath: ${{ github.workspace }}
      azureCredentials: ${{ secrets.AZURE_CREDENTIALS }}
      acrName: mytestacr
      acrUsername: ${{ secrets.REGISTRY_USERNAME }}
      acrPassword: ${{ secrets.REGISTRY_PASSWORD }}
      containerAppEnvironment: my-test-container-app-env
```

This will create a new Container App named `oryx-builder-<github-run-id>-<github-run-attempt>` in a new resource group named `<container-app-name>-rg` with a new Container App environment named `my-test-container-app-env`.

### Runtime stack provided

```yml
steps:
  - name: Build and deploy Container App
    uses cormacpayne/oryx-builder-action@main
    with:
      appSourcePath: ${{ github.workspace }}
      azureCredentials: ${{ secrets.AZURE_CREDENTIALS }}
      acrName: mytestacr
      acrUsername: ${{ secrets.REGISTRY_USERNAME }}
      acrPassword: ${{ secrets.REGISTRY_PASSWORD }}
      runtimeStack: 'dotnetcore:7.0'
```

This will create a new Container App named `oryx-builder-<github-run-id>-<github-run-attempt>` in a new resource group named `<container-app-name>-rg` where the runnable application image is using the .NET 7 runtime stack.
