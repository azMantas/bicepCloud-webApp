---
title: "Storage Accounts"
date: 2023-04-19T20:38:49+02:00
draft: false
---
# Storage Account

In this post, I'll show you how to create a Bicep template for an Azure Storage Account. This template comes with many customizable options, making it perfect for various projects. By default, I've made sure the parameters prioritize security. I'll explain each parameter in detail to help you understand how to customize them. By the end of this post, you'll know how to create a secure and customizable storage account using Bicep templates, tailored to your specific needs.

## bicep template

{{< ghcode "https://raw.githubusercontent.com/azMantas/bicepModules/main/storage/storageAccounts.bicep" >}}

## storage Account Name

`storageAccountName` This parameter represents the name of the Azure Storage Account being created. When creating a new storage account, it is important to choose a unique and meaningful name that adheres to the Azure Storage naming conventions. The storage account name acts as an identifier for the account and is used when accessing or managing the storage account within Azure.

Azure Storage Account naming conventions dictate that the name must:

- Be between 3 and 24 characters in length.
- Contain only lowercase letters and numbers.
- Be unique within an Azure. No two storage accounts can have the same name

## kind

In the context of Azure Storage Accounts, the kind property refers to the type of storage account that will be created. There are different kinds of storage accounts available, each designed to cater to specific use cases and requirements. The most popular kinds are:

`StorageV2` (General-purpose v2): This is the latest and most versatile storage account type. It is recommended for most use cases as it provides all the features of previous storage account types and supports blobs, files, queues, and tables. General-purpose v2 storage accounts offer multiple performance tiers, access tiers, and replication options. They also provide features such as Azure Blob Storage lifecycle management, Azure Files with Active Directory (AD) authentication, and large file shares.

`FileStorage` This type of storage account is specifically designed for storing files using Azure Files. It supports the Premium performance tier, offering low-latency and high transaction rates. However, `FileStorage` accounts do not support general-purpose workloads, as they do not support blobs, queues, or tables.

## sku Name

The `skuName` property in an Azure Storage Account refers to the SKU (Stock Keeping Unit) which represents a combination of performance tier and redundancy level. The SKU determines the pricing and capabilities of the storage account.Here's a brief explanation of the performance tiers and redundancy levels:

#### Performance tiers:

- Standard - Offers cost-effective storage with general-purpose capabilities, suitable for most workloads. It supports block blobs, page blobs, files, queues, and tables.  

- Premium - Provides high-performance storage with lower latency, designed for workloads requiring high throughput. It supports block blobs, page blobs, and files but does not support queues or tables.  

#### Redundancy levels:

 - Locally Redundant Storage (LRS) - Provides data replication within a single data center. LRS maintains three copies of your data, ensuring durability in case of hardware failures. 

- Zone-Redundant Storage (ZRS) - Replicates data across multiple availability zones within a region, providing higher durability and availability compared to LRS.  

- Geo-Redundant Storage (GRS) - Replicates data to a secondary region, providing better durability and availability in case of a regional outage or disaster. It maintains six copies of your data: three in the primary region and three in the secondary region.  

- Read-Access Geo-Redundant Storage (RA-GRS) - Offers the same replication as GRS with the added benefit of read access to the data in the secondary region. This allows for faster data access during a regional outage or disaster.  

When configuring the `skuName` property for a storage account, you need to specify the combination of performance tier and redundancy level you want to use. Each `skuName` has different pricing and capabilities, so it's essential to choose the one that best meets your performance, durability, and budget requirements.

## supports Https Traffic Only

`supportsHttpsTrafficOnly` is a property of an Azure Storage Account that determines whether the storage account enforces the use of secure communication over HTTPS. When this property is set to `true`, the storage account only allows HTTPS traffic, which ensures that all communication between clients and the storage service is encrypted and secure.

By enforcing HTTPS-only access, you can prevent potential security risks associated with unencrypted communication, such as data interception, tampering, and man-in-the-middle attacks. When the `supportsHttpsTrafficOnly` property is set to `true`, any requests made to the storage account over HTTP will be rejected, ensuring that data can only be accessed using secure communication channels.

In summary, enabling the `supportsHttpsTrafficOnly` property for an Azure Storage Account is a security best practice to protect your data during transit. It ensures that all communication with your storage account is encrypted and secure by only allowing HTTPS traffic.


## is Hns Enabled

 Hierarchical Namespace (HNS), also known as Azure Data Lake, is a cloud-based storage service from Microsoft. It is designed for big data analytics and can handle large amounts of structured and unstructured data. It offers scalable and cost-effective storage, allowing users to store, process, and analyze data efficiently and securely.

## access Tier

The `accessTier` property in an Azure Storage Account determines the performance and access cost characteristics of the stored data. Azure Storage provides two access tiers: `Hot` and `Cool`. These tiers are available for Block Blob storage and Azure Blob Storage accounts. The purpose of these access tiers is to enable you to store and manage your data based on its access patterns and optimize costs.

`Hot Access Tier` is designed for data that is accessed frequently or requires low latency. In this tier, access costs are lower, while storage costs are higher compared to the 'Cool' tier. The 'Hot' tier is suitable for use cases such as big data analytics, content delivery, backups, and data that is accessed and modified often.

`Cool Access Tier` is designed for data that is infrequently accessed and can tolerate slightly higher access latency. The storage costs in this tier are lower compared to the 'Hot' tier, but access costs are higher. The 'Cool' tier is suitable for use cases such as long-term backups, older media content, and data that is not accessed frequently but needs to be retained for compliance or business reasons.

When creating or modifying an Azure Storage Account, you can set the `accessTier` property to either `Hot` or `Cool` to define the default access tier for the account. Note that the access tier can also be set at the blob level, allowing you to optimize storage costs based on individual blob access patterns.

By understanding the access patterns of your data and choosing the appropriate access tier, you can optimize the costs associated with your Azure Storage Account while meeting your performance and latency requirements.

## minimum Tls Version

`minimumTlsVersion` This property specifies the minimum supported version of the Transport Layer Security (TLS) protocol for your Azure Storage Account. TLS is a cryptographic protocol that provides secure communication between clients and servers over a network. It is widely used to secure web traffic, and in the case of Azure Storage Accounts, it secures communication between clients and the storage service.

Setting a minimum TLS version helps ensure that only clients using a specific version or higher of TLS can access the storage account. This is important because older versions of TLS have known vulnerabilities and are considered less secure. By enforcing a higher minimum TLS version, you can protect your storage account from potential attacks that exploit these vulnerabilities.

The minimumTlsVersion property accepts the following values:

 - TLS1_0
 - TLS1_1
 - TLS1_2
  
For example, setting minimumTlsVersion to 'TLS1_2' in your storage account configuration ensures that only clients using TLS 1.2 or higher can access the storage account. This helps protect your storage account from potential attacks that target older, less secure versions of TLS.

It is important to consider the potential impact on your clients when setting the minimum TLS version. Older clients or software libraries may not support newer TLS versions, which could lead to compatibility issues. However, it is generally recommended to use the highest possible minimum TLS version that your clients can support, in order to ensure the security of your storage account.

## allow Shared Key Access

`allowSharedKeyAccess` This property controls whether the storage account allows authentication using shared keys. Shared keys are account-level keys used for authenticating requests made against the storage account. There are two types of shared keys: the primary key and the secondary key. These keys can be used to sign requests made to the storage account, allowing the caller to perform operations within the scope of the storage account.

In contrast, Azure Role-Based Access Control (RBAC) is a granular access control mechanism that allows you to assign permissions to users, groups, or applications in the context of Azure resources. 

By setting `allowSharedKeyAccess` to `false`, you disable shared key authentication for the storage account, ensuring that only Azure RBAC and Azure Active Directory based access is allowed. This enhances security, simplifies access management, and enables better auditing and monitoring of access to your storage account.

Advantages of using RBAC over shared keys (local authentication):

 - Granular permissions: RBAC allows you to assign specific permissions to users, groups, or applications, whereas shared keys grant full access to the storage account. This enables better access control and limits the potential damage caused by compromised keys.

 - Identity-based access: RBAC leverages Azure Active Directory (AAD) for authentication, allowing you to manage access based on users, groups, or applications. This simplifies access management and enhances security compared to shared keys, which require manual distribution and rotation.

- Auditing and monitoring: RBAC enables better auditing and monitoring of access to storage accounts, as each access request can be tied to a specific user, group, or application. With shared keys, it is difficult to track who made a particular request, as the same key might be used by multiple users or applications.

- Key rotation and management: RBAC eliminates the need to rotate and manage shared keys, reducing the administrative overhead and potential for human error. With shared keys, you must manually manage and rotate keys to maintain security.

## allow Blob Public Access

`allowBlobPublicAccess` This property controls whether public read access is allowed for the blob containers in your storage account. Public read access means that anonymous users can read the data stored in the blob containers without providing any authentication.

When you set `allowBlobPublicAccess` to `true`, you enable the option to make blob containers within the storage account publicly accessible. In this case, you can configure the public access level for each container individually. When you set `allowBlobPublicAccess` to `false`, public read access is disabled for all blob containers within the storage account, regardless of their individual access level settings. This ensures that access to the containers and their contents requires proper authentication.

In most scenarios, it is recommended to set `allowBlobPublicAccess` to `false` to protect your data from unauthorized access. If you need to provide public read access to specific blobs or containers, carefully evaluate the risks associated with exposing your data publicly.

## allow cross tenant replication

`allowCrossTenantReplication` is a property in Azure Storage Accounts that determines whether data in the storage account can be replicated across tenants. When enabled, it allows the storage account to participate in cross-tenant replication, which means that the data can be replicated to a secondary storage account located in a different Azure tenant. This can be useful for scenarios where you need to ensure data redundancy and disaster recovery across different organizations.

Cross-tenant replication helps organizations to achieve higher levels of data resiliency by providing the ability to store redundant copies of their data in different Azure tenants. This is particularly useful in scenarios where organizations have regulatory or compliance requirements to store data redundantly across separate legal entities or when multiple organizations collaborate and need to maintain synchronized copies of their data. By default, this property is set to `true`, which means that data in the storage account can be replicated across tenants.

It's important to note that enabling cross-tenant replication for a storage account can have implications on data privacy, security, and compliance. Before enabling this feature, you should carefully consider your organization's requirements and ensure that you have the necessary permissions and agreements in place to store data across different tenants.

## networkAcls - deny

The `defaultAction` property is part of the network access control settings in an Azure Storage Account. It specifies the default action to take when a request to access the storage account does not match any of the defined rules in the network access control list (ACL).

Setting the `defaultAction` to `Deny` means that by default, all incoming requests will be denied access to the storage account unless they meet specific criteria defined in the `ipRules` or `virtualNetworkRules`. This approach is a security best practice, as it follows the principle of least privilege, where access is granted only to the resources and actions explicitly allowed.

By using `Deny` as the default action, you can restrict access to your storage account and protect it from unauthorized access. You can then create specific rules to allow access from trusted IP addresses or virtual networks. This helps ensure that only authorized clients can access the storage account, reducing the potential attack surface and improving the overall security of your data.

## networkAcls - bypass

`AzureServices` is a term used within the context of Azure network access control lists (ACLs) when configuring the bypass property for a storage account. The bypass property is used to specify which types of traffic are allowed to bypass the network access control rules defined for the storage account.

When the bypass property is set to `AzureServices`, it means that certain Azure services can bypass the network ACLs and access the storage account without being restricted by the IP address or virtual network rules. This is useful in scenarios where you need to allow Azure services, such as Azure Monitor to access your storage account, while still enforcing network access control rules for other types of traffic.

It is important to note that enabling `AzureServices` bypass does not grant access to all Azure services randomly. The actual access is still controlled by the Azure RBAC and authentication mechanisms in place. By allowing the bypass for Azure services, you are permitting these services to communicate with your storage account without being blocked by the network-level restrictions.