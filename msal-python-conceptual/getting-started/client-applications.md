---
title: Client applications
description: How to instantiate client applications in MSAL Python.
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
---

# Client applications

## Instantiating an application

### Prerequisites

Before instantiating your app with MSAL Python:

1. Understand the types of client applications available - [Public Client and Confidential Client applications](/azure/active-directory/develop/msal-client-applications).
1. You'll need to [register the application](/azure/active-directory/develop/quickstart-register-app) with Microsoft Entra ID. From the registration page, you will need:
    - The application client identifier (a string in the form of a globally unique identifier (GUID))
    - The identity provider URL (the instance) and the sign-in audience for your application. These two parameters are collectively known as the **authority**.
    - If necessary, the tenant identifier (also a GUID) in case you are writing a line-of-business application scoped to just your organization (also known as a single-tenant application).
    - If you are building a confidential client app, you will need to create an application secret in the form of a string or certificate.
    - For web applications, you'll have also set the redirect URL that Microsoft Entra ID will use to return the code. For desktop applications you will need to add `http://localhost` if you're not relying on authentication brokers.

### Instantiating a public client application

Public client applications use the [`PublicClientApplication`](xref:msal.application.PublicClientApplication) class.

```python
app = msal.PublicClientApplication(
    "client_id",
    authority="authority",
    )
```

### Instantiating a confidential client application

Confidential client applications use the [`ConfidentialClientApplication`](xref:msal.application.ConfidentialClientApplication) class.

```python
app = msal.ConfidentialClientApplication(
    "client_id",
    authority="authority",
    client_credential="client_secret",
    )
```
