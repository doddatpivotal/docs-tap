# Create Azure Resources for Tanzu Application Platform

To install Tanzu Application Platform (commonly known as TAP) within the Azure ecosystem, 
you must create several Azure resources. Use this topic to learn how to create:

- An Azure Kubernetes Service (AKS) cluster to install Tanzu Application Platform.
- ACR repositories for the Tanzu Application Platform container images.

Creating these resources enables Tanzu Application Platform to use an IAM role 
bound to a Kubernetes service account for authentication, rather than the typical 
username and password stored in a Kubernetes secret strategy. 

This is important when using ACR because authenticating to ACR is a two-step process:

1. Retrieve a token using your Azure credentials.
1. Use the token to authenticate to the registry.

To increase security, the token has a lifetime of 12 hours. This makes storing 
it as a secret for a service impractical because it must be refreshed every 12 hours.

Using an IAM role on a service account mitigates the need to retrieve the token 
because it is handled by credential helpers within the services.

## <a id='prereqs'></a>Prerequisites

Before installing Tanzu Application Platform on Azure, you need:

- **An Azure subscription:**

    You must create all of your resources within
    an [Azure subscription](https://learn.microsoft.com/en-us/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing) and create an [Azure free account](https://azure.microsoft.com/en-us/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
  
- **Azure CLI:**

    To run CLI reference commands locally, you must [install the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).
    This topic uses Azure CLI to both query and configure resources in Azure, such as IAM roles.
    For more information, see [Azure CLI documentation](https://learn.microsoft.com/en-us/cli/azure/authenticate-azure-cli).


## <a id='azure-resource-group'></a>Create Azure Resource Group

1. Log in to Azure.

    ```console
    az login
    az account set --subscription SUBSCRIPTION-NAME
    ```

1. Create a resource group with the `az group create` command.

    ```console
    az group create --name myTAPResourceGroup --location eastus
    ```

## <a id='create-aks-cluster'></a>Create an AKS cluster

To create an AKS cluster, you can run the [az aks create](https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create) 
command with the `--enable-addons monitoring` and `--enable-msi-auth-for-monitoring` 
parameter to enable [Azure Monitor Container insights](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-overview) 
with managed identity authentication (preview). 

The following example creates a cluster named `tap-on-azure` with one node and 
enables a system-assigned managed identity:

```console
az aks create -g myTAPResourceGroup -n tap-on-azure --enable-managed-identity --node-count 6 --enable-addons monitoring --enable-msi-auth-for-monitoring --generate-ssh-keys --node-vm-size Standard_D4ds_v4 --kubernetes-version K8S-VERSION
```

Where `K8S-VERSION` is the compatible Kubernetes version that can be retrieved by running `az aks get-versions`.

After a few minutes, the command completes and returns JSON-formatted information about the cluster.

> **Note** When you create an AKS cluster, a second resource group is automatically 
created to store the AKS resources. 
For more information, see [Why are two resource groups created with AKS?](https://learn.microsoft.com/en-us/azure/aks/faq#why-are-two-resource-groups-created-with-aks)

## <a id='connect-aks-cluster'></a>Connect to the AKS cluster

To manage a Kubernetes cluster, use the Kubernetes command-line client, 
[kubectl](https://kubernetes.io/docs/reference/kubectl/). 
`kubectl` is already installed if you use Azure Cloud Shell.

1. Install `kubectl` locally by using the 
  [az aks install-cli](https://learn.microsoft.com/en-us/cli/azure/aks#az-aks-install-cli) command:

    ```console
    az aks install-cli
    ```

1. Configure `kubectl` to connect to your Kubernetes cluster by using the 
  [az aks get-credentials](https://learn.microsoft.com/en-us/cli/azure/aks#az-aks-get-credentials) command that:

    - Downloads credentials and configures the Kubernetes CLI to use them.
    - Uses `~/.kube/config`, the default location for the [Kubernetes configuration file](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/). 
    You can specify a different location for your Kubernetes configuration file by using the `--file` argument.

    ```console
    az aks get-credentials --resource-group myTAPResourceGroup --name tap-on-azure
    ```

## <a id='create-container-repos'></a>Create the container repositories

Azure Container Registry (ACR) does not require that the container repositories are already created. Repositories are created automatically when images are uploaded.

## <a id='enable-admin-account'></a>Enable registry admin account

To enable push and pull to your registries, you must enable the admin user account, which is created with each registry. Run the following command to enable the admin user account:

```console
az acr update -n $REGISTRY_NAME --admin-enabled true
```

There are two passwords created for each admin user account per registry. To retrieve the passwords, run the following for each registry:

```console
az acr credential show --name $REGISTRY_NAME --resource-group myTAPResourceGroup
```

Expect to see the following outputs:

```console
{
  "passwords": [
    {
      "name": "password",
      "value": YOUR-PASSWORD
    },
    {
      "name": "password2",
      "value": YOUR-PASSWORD-2
    }
  ],
  "username": ""
}
```

Export the username and password by running:

```console
export KP_REGISTRY_USERNAME=$REGISTRY_NAME
export KP_REGISTRY_PASSWORD=YOUR-PASSWORD
```

## <a id='next-steps'></a>Next steps

- [Install Tanzu Application Platform package and profiles on Azure](profile.hbs.md)
