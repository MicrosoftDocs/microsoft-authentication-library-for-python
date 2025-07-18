---
title: Overview of the Microsoft Authentication Library (MSAL) for Python
description: Get started with the Microsoft Authentication Library for Python to sign in users or apps with Microsoft identities."
author: SHERMANOUKO
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: article
ms.date: 04/24/2025
ms.author: shermanouko
ms.reviewer: dmwendia, rayluo
#Customer intent: 
---

# Microsoft Authentication Library (MSAL) for Python

The Microsoft Authentication Library (MSAL) for Python library enables you to sign in users or apps with Microsoft identities ([Microsoft Entra ID](https://azure.microsoft.com/services/active-directory/), [Microsoft Accounts](https://account.microsoft.com), and [Microsoft Entra ID](https://www.microsoft.com/security/business/identity-access/microsoft-entra-id) accounts). Using MSAL Python, you can acquire tokens from Microsoft Entra ID to call protected web APIs such as [Microsoft Graph](https://graph.microsoft.io/), other Microsoft APIs, or your own APIs.

## Prerequisites

- An Azure account with an active subscription. [Create a free account](https://signup.azure.com/).
- [Python 3.6+](https://www.python.org/downloads/).

## Install the package

Install the MSAL for Python package. You can find MSAL Python on [PyPI](https://pypi.org/project/msal/).

```Bash
pip install msal
```

## Identity concepts

MSAL Python is part of the [Microsoft identity platform](/entra/identity-platform/v2-overview) ecosystem. Familiarize yourself with the following concepts to effectively use MSAL Python to protect your applications and APIs:

- [Identity and access management](/entra/fundamentals/identity-fundamental-concepts)
- [Authentication and authorization](/entra/identity-platform/authentication-vs-authorization)
- [OAuth 2.0 and OpenID Connect (OIDC) in the Microsoft identity platform](/entra/identity-platform/v2-protocols)
- [Confidential and public client accounts in the Microsoft identity platform](/entra/identity-platform/msal-client-applications)
- [Security tokens](/entra/identity-platform/security-tokens)

## Usage scenarios

To use MSAL Python, register an application with the Microsoft identity platform. You'll need an Azure account with an active subscription. [Create a free account](https://signup.azure.com/) if you don't have one. You can register your app in a [customer tenant](/entra/external-id/customers/quickstart-tenant-setup) or [workforce tenant](/entra/identity-platform/scenario-web-app-sign-user-app-registration?tabs=python).

Applications can use MSAL Python to acquire tokens for accessing protected APIs. Different app types acquire tokens using different auth flows. The supported app types include desktop applications, web applications, web APIs, and applications running on devices that don't have a browser (such as IoT devices).

In MSAL Python, applications are categorized as follows:

- [Public client applications](https://datatracker.ietf.org/doc/html/rfc6749#section-2.1) (desktop and mobile). These types of apps cannot store app secrets securely.
- [Confidential client applications](https://datatracker.ietf.org/doc/html/rfc6749#section-2.1) (web apps, web APIs, and daemon applications). These type of apps securely store a secret registered with Microsoft Entra ID.

For more information, see the documentation on [public client and confidential client apps](/entra/identity-platform/msal-client-applications) and the [different app types and their auth flows](/entra/identity-platform/authentication-flows-app-scenarios) in the Microsoft identity platform.

After determining whether your application is a public or confidential client application, you can use MSAL Python to acquire tokens for different scenarios.

## Basic usage

Acquiring tokens with MSAL Python follows a three-step pattern. There will be some variations for different flows. If you would like to see them in action, download our [samples](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample).

1. MSAL relies on a clean separation between public client and confidential client applications. Therefore, create either a [`PublicClientApplication`](xref:msal.application.PublicClientApplication) or a [`ConfidentialClientApplication`](xref:msal.application.ConfidentialClientApplication) instance and reuse it during the lifecycle of your application. For example, for a public client application, the initialization code might look like this:

    ```python
    from msal import PublicClientApplication

    app = PublicClientApplication(
        "your_client_id",
        authority="https://login.microsoftonline.com/common")
    ```

    The authority value varies depending on the type of accounts you are signing-in and the kind of tenant your app is registered in. For example, to sign-in both work and personal Microsoft accounts provisioned in workforce tenants (Microsoft Entra ID) you would use `https://login.microsoftonline.com/common`. For customer accounts provisioned in customer tenants, your authority will take a form like `https://<subdomain>.ciamlogin.com`. For more information, see [token issuer documentation](/entra/identity-platform/access-tokens#validate-the-issuer).

1. Try and obtain the tokens from the cache first. The API model in MSAL provides you explicit control on how to utilize the token cache. While the caching part is technically optional, we highly recommend you to use it in your application. Using the cache you can ensure that you're not making any extra API calls and handle the token refresh automatically.

    ```python
    # initialize result variable to hole the token response
    result = None 
    
    # We now check the cache to see
    # whether we already have some accounts that the end user already used to sign in before.
    accounts = app.get_accounts()
    if accounts:
        # If so, you could then somehow display these accounts and let end user choose
        print("Pick the account you want to use to proceed:")
        for a in accounts:
            print(a["username"])
        # Assuming the end user chose this one
        chosen = accounts[0]
        # Now let's try to find a token in cache for this account
        result = app.acquire_token_silent(["User.Read"], account=chosen)
    ```

1. If there's no suitable token in the cache or you chose to skip the previous step, send a request to Microsoft Entra ID to get a token. There are different methods based on your client type and scenario, but for the purposes of the example we're showing how to use [`acquire_token_interactive`](xref:msal.application.PublicClientApplication.acquire_token_interactive), which prompts the user to provide their credentials.

   ```python
   if not result:
       # So no suitable token exists in cache. Let's get a new one from Azure AD.
       result = app.acquire_token_interactive(scopes=["User.Read"])
   if "access_token" in result:
       print(result["access_token"])  # Yay!
   else:
       print(result.get("error"))
       print(result.get("error_description"))
       print(result.get("correlation_id"))  # You may need this when reporting a bug
   ```

1. Save the code into a Python file locally, such as *msaltest.py*.
1. Run the code by executing `python .\msalpytest.py`. The following visual shows the sign-in experience for this example.

    ![Example of an app prompting the user to sign in with their account](./media/basic-pca-app-prompt.gif)

1. Once the authentication is completed and you closed the browser, you should be able to see the access token printed in the terminal.

## Best practices for a robust enterprise ready application

You can acquire a token for a protected Web API using MSAL Python. You also don't have to handle refreshing tokens yourself. However, to build robust, enterprise ready applications, you will need to do a bit more. For instance you'll want to:

- [Handle exceptions](/azure/active-directory/develop/msal-handling-exceptions?tabs=python), both when you acquire a token, but also when you call the protected Web API. In particular, if your application runs in a Microsoft Entra tenant where the tenant admins have set [Conditional Access](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki/Conditional-Access-and-Claims-Challenges) policies to enforce Multiple Factor Authentication (MFA), you will need to handle a Claim challenge.

- You might want to enable [Logging](/azure/active-directory/develop/msal-logging?tabs=python) to troubleshoot your application and help your users, while respecting their privacy and being compliant with GDPR.

## Samples

There are several samples you can use to get started with MSAL Python.

- Samples from the [library repository](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.22.0/sample). These samples demonstrate the different configurations and auth flows that you implement using MSAL Python.
- A single repository with [samples used in our documentation](https://github.com/Azure-Samples/ms-identity-docs-code-python). These samples have supporting documentation to help you build and replicate them from scratch.

## References

- MSAL Python library repository on [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-python)
- MSAL Python releases on [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-python/releases).

## See also

- [Instantiate your application](./getting-started/client-applications.md) using MSAL Python
- [Acquire tokens](./getting-started/acquiring-tokens.md) using MSAL Python
