---
title: "assign RBAC roles with bicep"
date: 2023-04-29
draft: false
tags: ["RBAC", "storageAccount"] 
---
In this blog post, we will walk through a step-by-step guide on how to assign Azure RBAC roles to Storage Accounts using Bicep. By the end of this post, you'll be equipped with the knowledge to create role assignments for your Azure resources. While this post will primarily focus on Azure Storage Accounts, the principles and techniques discussed can be easily adapted to suit other resources like KeyVault, AppServices and so on.

### bicep module to assign RBAC at the storage account level
In this example, we'll create a Bicep module that assigns RBAC roles to users, groups, or service principals for a specific Azure storage account. To start, we'll need to provide the name of the Azure storage account where we want to assign the RBAC role. Next, we'll specify the principal IDs and types for the AzureAD objects who will receive the role assignment. Finally, we'll need to provide the RBAC role definition ID to define the role we want to assign to our users.

```bicep
@allowed([ 'User', 'ServicePrincipal', 'Group' ])
param principalType string
param principalIds array
param roledefinition string
param storageAccountName string

resource targetResource 'Microsoft.Storage/storageAccounts@2022-09-01' existing = {
  name: storageAccountName
}

resource rbac 'Microsoft.Authorization/roleAssignments@2020-08-01-preview' = [for principal in principalIds: {
  name: guid(principal, roledefinition, targetResource.id)
  scope: targetResource
  properties: {
    roleDefinitionId: roledefinition
    principalId: principal
    principalType: principalType
  }
}]

```
While this template works well for assigning the same role to multiple users, real-world scenarios often require assigning multiple roles to the same principal. To accomplish this, we'll need to modify the template to support multiple role definitions rather than principal IDs.

```bicep
@allowed([ 'User', 'ServicePrincipal', 'Group' ])
param principalType string
param principalId string
param roledefinitions array
param storageAccountName string

resource targetResource 'Microsoft.Storage/storageAccounts@2022-09-01' existing = {
  name: storageAccountName
}

resource rbac 'Microsoft.Authorization/roleAssignments@2020-08-01-preview' = [for role in roledefinitions: {
  name: guid(principalId, role, targetResource.id)
  scope: targetResource
  properties: {
    roleDefinitionId: role
    principalId: principalId
    principalType: principalType
  }
}]

```
As this module is primarily focused on Azure Storage Accounts, it is worth noting that the same logic can be easily adapted and applied to other Azure resources with minor modifications to the template. For example, by adjusting the template, you can manage resources such as Azure Key Vaults.

```bicep
resource targetResource 'Microsoft.KeyVault/vaults@2023-02-01' existing = {
  name: keyVaultName
}
```
### generating unique GUID

Azure RBAC is a unique resource in the Azure ecosystem. Unlike other resources, it does not support human-readable names, and instead relies on GUIDs (Globally Unique Identifiers). When creating a new RBAC role through the Azure Portal, the platform automatically generates a GUID for each specific assignment.

The challenge arises when automating RBAC assignment using Bicep modules, as it is crucial to generate a new, unique GUID for each role while ensuring that the same GUID is used upon subsequent deployments. Additionally, it is important to avoid any conflicts where two different RBAC roles might share the same GUID.

To address this issue, a GUID is generated based on a combination of the principal ID, role ID, and resource ID. This approach ensures that each RBAC role is assigned a unique GUID, while also maintaining consistency across deployments. By using these three components, we can be confident that no two distinct RBAC roles will inadvertently share the same GUID, thus preventing potential conflicts and enhancing the overall reliability of the system.

### consuming module in the main.bicep file
First, we'll create a resource group and deploy an Azure storage account to it. Next, we'll use a Bicep module to assign multiple RBAC roles to the same Azure AD object, in this case, an AD group. Since this module has a resource group deployment scope, we'll also need to specify the scope where the Azure storage account is provisioned. Finally, we'll need to provide the name of the storage account. The easiest way to do this is to reference the `simpleStorage` module and use its output as the input.

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
    storageAccountName: simpleStorage.outputs.storageAccountName
  }
}

```

### known issues
- It is important to note that deployment may fail if you attempt to mix multiple principal types, such as ‘group’ and ‘ServicePrincipal’, within the same assignment. To avoid any potential issues, ensure that you separate principal types accordingly.
- At the time of writing, Microsoft has not disclosed the algorithm used to generate GUIDs. If assigned role has a different GUID, you may encounter the error: “role assignment already exists.” To resolve this issue, please manually remove the existing role assignment and redeploy the Bicep module