---
title: "Example deployment"
date: 2023-04-25T19:13:37+02:00
draft: false
weight: 100
---

In this blog post, I will walk you through the process of using the `storageAccount.bicep` template to create a resource in Azure. The module is preconfigured with default values for all parameters, simplifying the deployment process and reducing the likelihood of errors. However, if your specific use case demands customization or unique settings, you can effortlessly adjust these parameters to suit your requirements.
We'll start by deploying a basic Storage Account, and then gradually harness the power of the template to tailor the settings to our needs.

## deploy a simple Storage Account

Here will walk through the process of deploying an Azure Storage Account using a Bicep module. As with most Azure resources, the Storage Account must be deployed within a Resource Group — a logical container for resources deployed within an Azure subscription. To get started, we will create a new Resource Group that will house our Storage Account.

To provision the Storage Account, we need to provide at least two values: `projectName` and `environment`. These inputs are essential for generating a unique Storage Account name, ensuring that it stick to Azure's naming conventions and does not conflict with existing resources.

```bicep
targetScope = 'subscription'

param projectName string = 'bicepRules'
param environment string = 'tst'
param location string = 'westeurope'

resource storageResourceGroup 'Microsoft.Resources/resourceGroups@2022-09-01' = {
  name: 'storage-${projectName}-${environment}'
  location: location
}

module simpleStorage 'storageAccounts.bicep' = {
  scope: storageResourceGroup
  name: 'simple-storage-account'
  params: {
    projectName: projectName
    environment: environment
  }
}

output simpleStorageAccountName string = simpleStorage.outputs.storageAccountName

```
{{< tabs "deployStorageAccount" >}}
{{< tab "Azure CLI" >}} ```cli
az deployment create --location WestUS --template-file example.bicep
``` {{< /tab >}}
{{< tab "PowerShell" >}} 
```Powershell
$params = @{
  location = 'westeurope'
  templateFile = example.bicep
}
new-azsubscriptionDeployment @params -verbose
```
 {{< /tab >}}
{{< /tabs >}}


## configure Storage Account

In this example, we will once again use the `storageAccount.bicep` module; however, we will adjust some of the predefined parameters to satisfy particular needs. The Storage Account will be deployed within the same Resource Group as the previously created 'simple storage account'. Instead of using the default naming convention, we will generate the Storage Account's name based on a hardcoded string, `datalake`.

Furthermore, we will configure this Storage Account to serve as a Data Lake, which is a scalable and cost-effective solution for storing and analyzing large volumes of structured and unstructured data. To enhance the organization and management of data within the Data Lake, we will create two separate containers. This will allow for better data partitioning and access control.

Lastly, we will enable Azure Defender for Storage, an advanced threat protection service that monitors and safeguards the Storage Account against potential security threats.

This example demonstrates how the `storageAccount.bicep` module can be customized to meet specific requirements, showcasing the flexibility and adaptability of the module for various use cases.

```bicep
targetScope = 'subscription'

param projectName string = 'bicepRules'
param environment string = 'tst'
param location string = 'westeurope'

resource storageResourceGroup 'Microsoft.Resources/resourceGroups@2022-09-01' = {
  name: 'storage-${projectName}-${environment}'
  location: location
}

module configParams 'storageAccounts.bicep' = {
  scope: storageResourceGroup
  name: 'config-storage-account'
  params: {
    projectName: 'datalake'
    environment: environment
    isHnsEnabled: true
    blobServicesContainers: [
      'PDF'
      'Incoming'
    ]
    advancedThreatProtectionEnabled: true
  }
}

output storageAccountName string = configParams.outputs.storageAccountName
output storageAccountUrl string = configParams.outputs.storageAccountDfsUrl
```
## how to assign RBAC?

Here we will demonstrate how to assign the 'Reader' and 'Blob Storage Reader' RBAC roles to an Azure AD group using the 'roleAssignments-roleDefinitions.bicep' module. Assigning these roles ensures that members of the Azure AD group have appropriate access to the Storage Account, enabling them to read its properties and metadata as well as access the Blob Storage content.

To accomplish this, we will first need to specify the Resource Group where the Storage Account is located. This allows the Bicep module to target the correct resource for the role assignments. Following this, we must provide several parameters to the 'roleAssignments-roleDefinitions.bicep' module:

- `StorageAccountName` The unique name of the Storage Account to which the roles will be assigned. In this example, we will dynamically fetch `datalake` storage account name.
{{< hint warning >}}
To dynamically retrieve a value from a storage account, it is important to ensure that the storage account deployment is located in the same Bicep file{{< /hint >}}
- `RoleDefinitions` The ID of the specific role, such as 'Reader' or 'Blob Storage Reader', that will be granted to the Azure AD group.
- `PrincipalID` The ID of the  Azure AD group that will receive the assigned roles.
  
By providing these parameters, the `roleAssignments-roleDefinitions.bicep` module can assign the specified roles to the designated Azure AD group. This ensures that the group members have the necessary permissions to access and interact with the Storage Account, while also maintaining the principle of least privilege to enhance security.

```bicep
targetScope = 'subscription'

param projectName string = 'bicepRules'
param environment string = 'tst'
param location string = 'westeurope'

resource storageResourceGroup 'Microsoft.Resources/resourceGroups@2022-09-01' = {
  name: 'storage-${projectName}-${environment}'
  location: location
}

module assignMultipleRoles 'roleAssignments-roleDefinitions.bicep' = {
  scope: storageResourceGroup
  name: 'rbac-multiple-roles'
  params: {
    principalId: 'e34ed234-5c06-4449-ad5a-c86b227adb21'
    principalType: 'Group'
    roledefinitions: [
      '/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7'
      '/providers/Microsoft.Authorization/roleDefinitions/ba92f5b4-2d11-453d-a403-e96b0029c9fe'
    ]
    storageAccountName: configParams.outputs.storageAccountName
  }
}
```

## create multiple Storage Accounts

Here, we will demonstrate how to deploy multiple Storage Accounts using the same 'storageAccount.bicep' template. This approach enables us to efficiently create multiple Storage Accounts with a single Bicep template, streamlining the deployment process and reducing the potential for errors.

First, we will generate a list of unique names for the Storage Accounts, which will be stored in a variable named `multipleStorageAccounts`. Following this, a loop will be employed to iterate through each name within the `multipleStorageAccounts` variable. During each iteration, the Bicep template will establish a Storage Account corresponding to the given name. In this particular example, a total of 15 Storage Accounts will be created, representing each name in the `multipleStorageAccounts` list. This ensures that a unique Storage Account name is generated based on the provided `projectName` for every iteration.

```bicep
targetScope = 'subscription'

param projectName string = 'bicepRules'
param environment string = 'tst'
param location string = 'westeurope'

var multipleStorageAccounts = [
  'Olivia'
  'William'
  'Sophia'
  'James'
  'Isabella'
  'Ethan'
  'Mia'
  'Alexander'
  'Ava'
  'Michael'
  'Emily'
  'Daniel'
  'Grace'
  'Benjamin'
  'Charlotte'
]

resource storageResourceGroup 'Microsoft.Resources/resourceGroups@2022-09-01' = {
  name: 'storage-${projectName}-${environment}'
  location: location
}

module multipleStorageAccount 'storageAccounts.bicep' = [for name in multipleStorageAccounts: {
  name: 'multiple-${name}'
  scope: storageResourceGroup
  params:{
    projectName: name
    environment: environment
  }
}]
```
## Deploy Storage Accounts to a dedicated Resource Group

```bicep

targetScope = 'subscription'

param environment string = 'tst'
param location string = 'westeurope'

var multipleStorageAccountV2 = [
  'Sophie'
  'Jacob'
  'Avery'
]

resource indexResourceGroup 'Microsoft.Resources/resourceGroups@2022-09-01' = [for (name, index) in multipleStorageAccountV2: {
  name: 'storage-${name}-${environment}'
  location: location
}]

module indexStorage 'storageAccounts.bicep' = [for (name, index) in multipleStorageAccountV2: {
  name: 'multiple-${name}'
  scope: indexResourceGroup[index]
  params:{
    projectName: name
    environment: environment
  }
}]

```