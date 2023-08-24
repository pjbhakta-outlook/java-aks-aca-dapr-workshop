Until now, you have use the connection string of Azure Service Bus and the master key of Cosmos DB to connect to these services. First you put them in the environment variables of the Dapr Component. Then you stored them in Azure Key Vault and referenced them in the Dapr Component using the secret store. This is a good practice for non-Azure services. However, for [Azure services that support Azure AD authentication](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/services-id-authentication-support), you should use [Managed Identities](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) to access them. With secrets in the secret store (i.e. Key Vault), you still have to manage the connection string of service bus and the master key of Cosmos DB, and their rotation. Even more, the client secret of the Service Principal used to access Key Vault is stored in a [platform-managed Kubernetes secret](https://learn.microsoft.com/en-us/azure/container-apps/dapr-overview?tabs=bicep1%2Cyaml#using-platform-managed-kubernetes-secrets)and is not rotated automatically. It is recommended to use managed identity to access key vault information.

To solve this problem, you will use [Managed Identities](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) that eliminate the need for developers to manage credentials. They provide an automatically managed identity in Azure Active Directory (Azure AD) that can be used to authenticate to [Azure services that support Azure AD authentication](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/services-id-authentication-support), like Azure Key Vault. Managed Identities can be assigned to Azure resources like Azure Container Apps, Azure Functions, Azure Virtual Machines, etc.

There are [two types of Managed Identities](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview#managed-identity-types) that can be assigned to a container app:
- `System-assigned Managed Identity` (SMI): a managed identity that is directrly enable on the container app. It is tied to the lifecycle of the container app and cannot be used by any other resource.
- `User-assigned Managed Identity` (UMI): a managed identity that is created as a standalone Azure resource. It can be assigned to one or more container apps and Azure resources. It is not tied to the lifecylce of a single container app as it is a shared resource.

In this assignment you will use a `User-assigned Managed Identity` (UMI) to access the Azure Container Registry (ACR) and Azure Service Bus. You will use a `System-assigned Managed Identity` (SMI) to access Azure Key Vault, and another one to access Azure Cosmos DB. You are going to use [Azure built-in roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles) and assign them to SMI and/or UMI. This assignment ends with a discussion on the choice between SMI and UMI.