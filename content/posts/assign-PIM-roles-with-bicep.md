---
title: "assign PIM roles with Bicep"
date: 2023-05-07T17:51:31+02:00
draft: false
tags: ["RBAC", "PIM"] 

---

Azure Privileged Identity Management (PIM) is a Azure service that enables Just In Time (JIT) access to Azure AD and RBAC roles. PIM allows creating role assignments that make users or groups eligible for specific roles without permanently granting them access. Instead, they can request the role when required. Please note that using Azure PIM requires Azure AD Premium P2 licenses.  
By adopting Azure PIM and incorporating organizations can enhance their security posture, improve governance, and maintain better control over access to sensitive resources. 

## create a bicep template 

The template itself is quite straightforward, but the `Microsoft.Authorization/roleEligibilityScheduleRequests` resource provider has several interesting properties:
- `name` the PIM assignment name is required to be a GUID. In contrast to RBAC assignments, this GUID is non-idempotent 
- `principalId`  object ID of the user or group you wish to assign the role to by navigating to the user or group within Azure Active Directory. Specifically, you'll need to find the Object ID field. 
- `startTime` utilize the `utcNow()` Bicep function to activate the role immediately after the deployment has successfully completed 
- `expiration` the duration of the role eligibility is measured in days. In this example, the role expires in 180 days, and users will no longer be able to activate the RBAC role after that time. 
- `roleDefinitionId` the full Id of role definition. 
- `requestType` I've discovered that `AdminRenew` is the most suitable option for my use cases. When the PIM role is nonexistent, it will create a new one, and if the role already exists, it will extend the expiration date. If a user has activated the role and the template is redeployed, the RBAC assignment remains active. 

The full template to assign a PIM role at the subscription scope looks like this: 

```bicep
targetScope = 'subscription'

param utc string = utcNow()
param AzureADGroupId string
param roleDefinitionId string
param roleGUID string = newGuid()

resource pim 'Microsoft.Authorization/roleEligibilityScheduleRequests@2020-10-01-preview' = {
  name: roleGUID
  properties: {
    roleDefinitionId: roleDefinitionId
    principalId: AzureADGroupId
    requestType: 'AdminRenew'
    scheduleInfo: {
      startDateTime: utc
      expiration: {
        type: 'AfterDuration'
        duration: 'P180D'
      }
    }
  }
}

output rbacId string = pim.id 
```
use azure portal to activate newly assigned role to verify that everything functions as expected. 

## known issues 

- After assigning the PIM role, we can begin configuring its settings, such as maximum activation duration, notifications, MFA, and approval requests. Unfortunately, I have not yet found a way to configure these settings via Bicep.  