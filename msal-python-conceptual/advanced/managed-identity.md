---
title: Using Managed Identity
description: Learn how to use Managed Identity with Microsoft Authentication Library (MSAL) for Python.
author: SHERMANOUKO

ms.service: msal
ms.subservice: msal-python
ms.topic: concept-article
ms.date: 06/25/2024
ms.author: shermanouko
ms.reviewer: rayluo, dmwendia
#Customer intent: 
---

# Using Managed Identity

[Managed identity](/entra/identity/managed-identities-azure-resources/overview) enables developers to remove the need to store, rotate, and otherwise handle credentials for their applications running in the Microsoft Azure cloud. Microsoft Authentication Library (MSAL) for Python supports using authentication with managed identity on supported Azure workloads.

>[!NOTE]
>Managed Identity in MSAL Python is supported since version [1.29.0](https://pypi.org/project/msal/1.29.0/) of the [`msal` package](https://pypi.org/project/msal/).

MSAL Python supports acquiring tokens through the managed identity service when used with applications running inside Azure infrastructure, such as:

- [Azure App Service](https://azure.microsoft.com/products/app-service/) (API version `2019-08-01`)
- [Azure VMs](https://azure.microsoft.com/free/virtual-machines/)
- [Azure Arc](/azure/azure-arc/overview)
- [Azure Cloud Shell](/azure/cloud-shell/overview)
- [Azure Service Fabric](/azure/service-fabric/service-fabric-overview)
- [Azure ML](/azure/machine-learning/how-to-identity-based-service-authentication)

For a complete list, refer to [Azure services that can use managed identities to access other services](/azure/active-directory/managed-identities-azure-resources/managed-identities-status).

## Which SDK to use - Azure SDK or MSAL?

MSAL libraries provide lower level APIs that are closer to the OAuth2 and OIDC protocols.

Both MSAL Python and Azure SDK allow to acquire tokens via managed identity. Internally, Azure SDK uses MSAL Python, and it provides a higher-level API via its [`DefaultAzureCredential`](/python/api/azure-identity/azure.identity.defaultazurecredential) and [`ManagedIdentityCredential`](/python/api/azure-identity/azure.identity.ManagedIdentityCredential) abstractions.

If your application already uses one of the SDKs, continue using the same SDK.

- Use Azure SDK if you are writing a new application and plan to call other Azure resources, as this SDK provides a better developer experience by allowing the app to run on private developer machines where managed identity doesn't exist.
- Use MSAL if you need to call other downstream web APIs like Microsoft Graph or your own web API.

## How to use managed identities

There are two types of managed identities available to developers - **system-assigned** and **user-assigned**. You can learn more about the differences in the [Managed identity types](/azure/active-directory/managed-identities-azure-resources/overview#managed-identity-types) article. MSAL Python supports acquiring tokens with both. MSAL Python logging allows to keep track of requests and related metadata.

Prior to using managed identities from MSAL Python, developers must enable them for the resources they want to use through Azure CLI or the Azure Portal.

## Examples

In both system- and user-assigned identities, developers need to use <xref:msal.managed_identity.ManagedIdentityClient> to access managed identities.

### System-assigned managed identities

System-assigned managed identities can be used by instantiating <xref:msal.managed_identity.SystemAssignedManagedIdentity> and passing to <xref:msal.managed_identity.ManagedIdentityClient>.

>[!NOTE]
>You need to include a `http_client` reference, which can be set to `requests.Session()`. This enables MSAL to maintain a pool of connections to the IMDS endpoint.

You can specify the target resource scope when calling [`acquire_token_for_client`](xref:msal.managed_identity.ManagedIdentityClient.acquire_token_for_client).

```python
import msal
import requests

managed_identity = msal.SystemAssignedManagedIdentity()

global_app = msal.ManagedIdentityClient(managed_identity, http_client=requests.Session())

result = global_app.acquire_token_for_client(resource='https://vault.azure.net')

if "access_token" in result:
    print("Token obtained!")
```

>[!IMPORTANT]
>You need to enable a system-assigned identity for the resource where the Python code runs; otherwise, no token will be returned.

### User-assigned managed identities

User-assigned managed identities can be used by instantiating <xref:msal.managed_identity.UserAssignedManagedIdentity> and passing to <xref:msal.managed_identity.ManagedIdentityClient>. You will need to specify the **one of the following**:

- Client ID (`client_id`)
- Resource ID (`resource_id`)
- Object ID (`object_id`)

>[!NOTE]
>You need to include a `http_client` reference, which can be set to `requests.Session()`. This enables MSAL to maintain a pool of connections to the IMDS endpoint.

You can specify the target resource scope when calling [`acquire_token_for_client`](xref:msal.managed_identity.ManagedIdentityClient.acquire_token_for_client).

```python
import msal
import requests

managed_identity = msal.UserAssignedManagedIdentity(client_id='YOUR_CLIENT_ID')

global_app = msal.ManagedIdentityClient(managed_identity, http_client=requests.Session())

result = global_app.acquire_token_for_client(resource='https://vault.azure.net')

if "access_token" in result:
    print("Token obtained!")
```

>[!NOTE]
>MSAL Python's [built-in managed identity sample](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.29.0/sample/managed_identity_sample.py#L38-L42) showcases how user-assigned managed identity can be inferred from environment variables. It's an advanced usage pattern that can be used instead of explicit definition of the client ID in code.

>[!IMPORTANT]
>You need to attach a user-assigned identity for the resource where the Python code runs; otherwise, no token will be returned. If an incorrect identifier is used for the user-assigned managed identity, no token will be returned as well.

## Caching

By default, MSAL Python supports in-memory caching.

>[!IMPORTANT]
>MSAL Python also supports cache extensibility for managed identity, so that you may persist the token cache on disk. This can be useful if you are writing a command-line script and a few other limited scenarios. We **do not recommend** sharing managed identity token cache among multiple machines as this can result in unexpected access behaviors for users of the cache. A token acquired for a node/machine, if cached in a distributed cache, can be used for another machine for which it is not intended.
