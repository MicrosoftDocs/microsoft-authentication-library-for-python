---
title: Client applications
description: "How to instantiate client applications in MSAL Python."
author: SHERMANOUKO
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: how-to
ms.date: 04/24/2025
ms.author: shermanouko 
ms.reviewer: dmwendia, rayluo
#Customer intent: 
---

# Client applications

Microsoft Authentication Library (MSAL) Python supports two types of client applications: public client applications and confidential client applications. The client types are distinguished by their ability to authenticate securely with the authorization server and to hold sensitive, identity proving information so that it can't be accessed or known to a user within the scope of its access. This article focuses on how to initialize these applications using MSAL Python.

## Prerequisites

Understand the basic concepts of [public and confidential client applications](/entra/identity-platform/msal-client-applications).

## Instantiate an application

You require an app registration to initiate an app. [Register an app](/entra/identity-platform/quickstart-register-app) in the Microsoft Entra admin center. From the registration page, you need:

- The application client identifieR. This is a string in the form of a GUID.
- The identity provider URL (the instance) and the sign-in audience for your application. These two parameters are collectively known as the *authority*.
- If necessary, the tenant identifier (also a GUID) in case you're writing a line of business application scoped to just your organization (also known as a single-tenant application).
- If you're building a confidential client app, create an application secret in the form of a string or certificate.
- For web applications, set the redirect URL that Microsoft Entra ID uses to return the code. For desktop applications, add `http://localhost` if you're not relying on authentication brokers.

## Instantiate a public client application

Public client applications use the [`PublicClientApplication`](xref:msal.application.PublicClientApplication) class.

```python
import msal

app = msal.PublicClientApplication(
    "client_id",
    authority="authority",
    )
```

## Instantiate a confidential client application

Confidential client applications use the [`ConfidentialClientApplication`](xref:msal.application.ConfidentialClientApplication) class.

```python
import msal

app = msal.ConfidentialClientApplication(
    "client_id",
    authority="authority",
    client_credential="client_secret",
    )
```

## Caching

When you instantiate a client application, there are two parameters you can use to define your caching preferences. These parameters are: *token_cache* and *http_cache*.

- *token_cache* sets the token cache used by the client application instance. By default, an in-memory cache is created and used. For more information, see [token caching in MSAL Python](../advanced/msal-python-token-cache-serialization.md).
- *http_cache* is available in MSAL Python version 1.16+. This automatically caches some finite number of nontoken http responses, so that long-lived *PublicClientApplication* and *ConfidentialClientApplication* instances would be more performant and responsive in some situations. If the *http_cache* parameter isn't provided, MSAL uses an in-memory dict. If your app is a command-line app (CLI), you would want to persist your *http_cache* across different CLI runs. For more information, see the [reference guide](/python/api/msal/msal.application.clientapplication).

## Next steps

[Acquire tokens for your app](acquiring-tokens.md).
