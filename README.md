---
services: azure-resource-manager
platforms: dotnet
author: devigned
---

# Manage Azure resources and resource groups with .NET

This sample explains how to manage your
[resources and resource groups in Azure](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/#resource-groups)
using the Azure .NET SDK.

**On this page**

- [Run this sample](#run)
- [What is program.cs doing?](#example)
    - [List resource groups](#list-groups)
    - [Create a key vault in the resource group](#create-resource)
    - [List resources within the group](#list-resources)
    - [Export the resource group template](#export)
    - [Delete a resource group](#delete-group)

<a id="run"></a>
## Run this sample

1. If you don't have it, install the [.NET Core SDK](https://www.microsoft.com/net/core).

1. Clone the repository.

    ```
    git clone https://github.com/Azure-Samples/resource-manager-dotnet-resources-and-groups.git
    ```

1. Install the dependencies.

    ```
    dotnet restore
    ```

1. Create an Azure service principal either through
    [Azure CLI](https://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal-cli/),
    [PowerShell](https://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal/)
    or [the portal](https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/).

1. Add these environment variables to your .env file using your subscription id and the tenant id, client id and client secret from the service principle that you created. 

    ```
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export AZURE_CLIENT_SECRET={your client secret}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    ```

1. Run the sample.

    ```
    dotnet run
    ```

<a id="example"></a>
## What is program.cs doing?

The sample walks you through several resource and resource group management operations.
It starts by setting up a ResourceManagementClient object using your subscription and credentials.

```csharp
// Build the service credentials and Azure Resource Manager clients
var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, secret);
var resourceClient = new ResourceManagementClient(serviceCreds);
resourceClient.SubscriptionId = subscriptionId;
```

<a id="list-groups"></a>
### List resource groups

List the resource groups in your subscription.

```csharp
resourceClient.ResourceGroups.List();
```

<a id="create-resource"></a>
### Create a key vault in the resource group

```csharp
var keyVaultParams = new GenericResource{
    Location = westus,
    Properties = new Dictionary<string, object>{
        {"tenantId", tenantId},
        {"sku", new Dictionary<string, object>{{"family", "A"}, {"name", "standard"}}},
        {"accessPolicies", Array.Empty<string>()},
        {"enabledForDeployment", true},
        {"enabledForTemplateDeployment", true},
        {"enabledForDiskEncryption", true}
    }
};
var keyVault = resourceClient.Resources.CreateOrUpdate(
    resourceGroupName,
    "Microsoft.KeyVault",
    "",
    "vaults",
    "azureSampleVault",
    "2015-06-01",
    keyVaultParams);
```

<a id="list-resources"></a>
### List resources within the group

```csharp
resourceClient.ResourceGroups.ListResources(resourceGroupName);
```

<a id="export"></a>
### Export the resource group template

You can export the resource group as a template and then use that
to [deploy your resources to Azure](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-template-deployment/).

```csharp
var exportResult = resourceClient.ResourceGroups.ExportTemplate(
    resourceGroupName, 
    new ExportTemplateRequest{ 
        Resources = new List<string>{"*"}
    });
```

<a id="delete-group"></a>
### Delete a resource group

```csharp
resourceClient.ResourceGroups.Delete(resourceGroupName);
```