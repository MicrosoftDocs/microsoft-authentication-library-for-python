---
title: Using MSAL Python with Authentication Brokers on macOS
description: "Using authentication brokers on macOS enables you to simplify how your users authenticate with Microsoft Entra ID from your application, as well as take advantage of advanced functionality such as token binding, protecting any issued tokens from exfiltration and misuse."
author: localden
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 09/06/2024
ms.author: ddelimarsky
ms.reviewer: shermanouko, rayluo
---

# Using MSAL Python with Authentication Brokers on macOS

>[!NOTE]
>macOS authentication broker support is introduced with `msal` version 1.31.0.

Using authentication brokers on macOS enables you to simplify how your users authenticate with Microsoft Entra ID from your application, as well as take advantage of advanced functionality such as **token binding**, protecting any issued tokens from exfiltration and misuse.

Authentication brokers are **not** pre-installed on macOS but are applications developed by Microsoft, such as [Company Portal](/mem/intune/apps/apps-company-portal-macos). These applications are usually installed when a macOS computer is enrolled in a company's device fleet via an endpoint management solution like [Microsoft Intune](/mem/intune/fundamentals/what-is-intune). To learn more about Apple device set up with the Microsoft Identity Platform, refer to [Microsoft Enterprise SSO plug-in for Apple devices](/entra/identity-platform/apple-sso-plugin).

## Usage

To use the broker, you will need to install the broker-related packages in addition to the core MSAL from PyPI:

```bash
pip install msal[broker]>=1.31,<2
```

>[!IMPORTANT]
>If broker-related packages are not installed and you will try to use the authentication broker, you will get an error: `ImportError: You need to install dependency by: pip install "msal[broker]>=1.31,<2"`.

Typically, on macOS your [public client](/entra/identity-platform/msal-client-applications) Python applications would [acquire tokens](../getting-started/acquiring-tokens.md) via the system browser. To use authentication brokers installed on a macOS system instead, you will need to pass an additional argument in the `PublicClientApplication` constructor - `enable_broker_on_mac`:

```python
from msal import PublicClientApplication
 
app = PublicClientApplication(
    "CLIENT_ID",
    authority="https://login.microsoftonline.com/common",
    enable_broker_on_mac =True)
```

>[!IMPORTANT]
>If you are writing a cross-platform application, you will also need to use `enable_broker_on_windows`, as outlined in the [Using MSAL Python with Web Account Manager](wam.md) article.

In addition to the constructor change, your application needs to support broker-specific redirect URIs. For _unsigned_ applications, the URI is:

```text
msauth.com.msauth.unsignedapp://auth 
```

For signed applications, the redirect URI should be:

```text
msauth.BUNDLE_ID://auth
```

If the redirect URIs are not correctly set in the app configuration within the Entra portal, you will receive error like this: 

```text
Error detected... 
tag=508170375
context=AADSTS50011 Description: (pii), Domain: MSAIMSIDOAuthErrorDomain.Error was thrown in location: Broker 
errorCode=-51411 
status=Response_Status.Status_Unexpected 
```

Once configured, you can call `acquire_token_interactive` to acquire a token.

```python
result = app.acquire_token_interactive(["User.ReadBasic.All"],
                    parent_window_handle=app.CONSOLE_WINDOW_HANDLE)
```

The `parent_window_handle` parameter is required even though on macOS it is not used. For GUI applications, the login prompt location will be determined ad-hoc and currently cannot be bound to a specific window.

>[!NOTE]
>The authentication broker handles refresh and access token caching. You do not need to set up custom caching.
