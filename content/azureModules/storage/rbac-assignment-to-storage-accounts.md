---
title: "RBAC assignments"
date: 2023-04-19T20:38:49+02:00
draft: false
weight: 20

---

# Implementing RBAC for Azure Storage Accounts

 To streamline the process of assigning RBAC roles to Azure Storage Accounts, we will utilize Bicep modules. These modules are designed to handle role assignment with ease, making it simpler to manage access rights across your environment. To assign an RBAC role to a storage account, you will need to provide the following information:

- Storage Account Name: The unique name of the target storage account to which you want to assign the role.
- Role Definition ID: The unique identifier of the predefined or custom role you want to assign.
- Object ID: The unique identifier of the user, Azure Active Directory (AAD) group, or Enterprise Application to which the role will be assigned.

I have developed two distinct modules to enhance the flexibility and functionality of managing access control in Azure environments. The first module allows for the assignment of RBAC roles to multiple principal IDs, while the second module supports the implementation of multiple role definitions.


## assign multiple users

In situations where you need to assign the same RBAC role, such as 'Storage Blob Data Reader', to multiple users, this solution proves to be most efficient. To accomplish this, simply provide an array of object IDs, and the template will handle the rest. 

{{< ghcode "https://raw.githubusercontent.com/azMantas/bicepModules/main/storage/roleAssignments-principals.bicep" >}}


## assign multiple role definitions

In scenarios where you need to assign multiple roles to a single principal, this solution is highly effective. To achieve this, simply provide an array of role definitions, and the template will take care of the rest

{{< ghcode "https://raw.githubusercontent.com/azMantas/bicepModules/main/storage/roleAssignments-roleDefinitions.bicep" >}}


## RBAC resource name (GUID)

Azure RBAC (Role-Based Access Control) is a unique resource in the Azure ecosystem. Unlike other resources, it does not support human-readable names, and instead relies on GUIDs (Globally Unique Identifiers) for identification. When creating a new RBAC role through the Azure Portal, the platform automatically generates a GUID for each specific assignment.

The challenge arises when automating RBAC assignment using Bicep modules, as it is crucial to generate a new, unique GUID for each role while ensuring that the same GUID is used upon subsequent deployments. Additionally, it is important to avoid any conflicts where two different RBAC roles might share the same GUID.

To address this issue, a GUID is generated based on a combination of the principal ID, role ID, and resource ID. This approach ensures that each RBAC role is assigned a unique GUID, while also maintaining consistency across deployments. By using these three components, we can be confident that no two distinct RBAC roles will inadvertently share the same GUID, thus preventing potential conflicts and enhancing the overall reliability of the system.

{{< hint warning >}}
At the time of writing, Microsoft has not disclosed the algorithm used to generate GUIDs. If assigned role has a different GUID, you may encounter the error: "role assignment already exists." To resolve this issue, please manually remove the existing role assignment and redeploy the Bicep module.{{< /hint >}}

# known issues

It is important to note that deployment may fail if you attempt to mix multiple principal types, such as 'group' and 'ServicePrincipal', within the same assignment. To avoid any potential issues, ensure that you separate principal types accordingly.