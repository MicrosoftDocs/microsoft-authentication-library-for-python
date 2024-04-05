---
title: Overview of the Microsoft Authentication Library (MSAL) for Python
description: Get started with the Microsoft Authentication Library for Python to sign in users or apps with Microsoft identities."
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
---

# Microsoft Authentication Library (MSAL) for Python

The Microsoft Authentication Library (MSAL) for Python library enables you to sign in users or apps with Microsoft identities ([Microsoft Entra ID](https://azure.microsoft.com/services/active-directory/), [Microsoft Accounts](https://account.microsoft.com), and [Azure AD B2C](https://azure.microsoft.com/services/active-directory-b2c/) accounts). Using MSAL Python, you can acquire tokens from Microsoft Entra ID to call protected web APIs such as [Microsoft Graph](https://graph.microsoft.io/), other Microsoft APIs, or your own APIs.

## Prerequisites

- An Azure account with an active subscription. [Create a free account](https://signup.azure.com/).
- [Python 3.6+](https://www.python.org/downloads/).

## Install the package

Install the MSAL for Python package. You can find MSAL Python on [Python Package Index (PyPI)](https://pypi.org/project/msal/).

```bash
pip install msal
```

## Setting up

Before using MSAL Python, make sure to [register your application](/azure/active-directory/develop/quickstart-v2-register-an-app) with the Microsoft identity platform. You will need to take note of your **client ID** and **tenant ID**.

>[!IMPORTANT]
> When registering the application, make sure that you set up **redirect URLs** within the **Authentication** blade. Redirect URLs vary depending on the target platform.
>
>![Screenshot showing redirect URLs in Azure Portal](./media/redirect-urls.png)
>
> For desktop and mobile applications, make sure you add `http://localhost` as redirect URL if you do not rely on authentication brokers.

## Basic usage

Acquiring tokens with MSAL Python follows a three-step pattern. There will be some variations for different flows. If you would like to see them in action, download our [samples](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample).

1. MSAL relies on a clean separation between [public client and confidential client applications](https://tools.ietf.org/html/rfc6749#section-2.1). Therefore, create either a [`PublicClientApplication`](xref:msal.application.PublicClientApplication) or a [`ConfidentialClientApplication`](xref:msal.application.ConfidentialClientApplication) instance and reuse it during the lifecycle of your application. For example, for a public client application, the initalization code might look like this:

    ```python
    from msal import PublicClientApplication

    app = PublicClientApplication(
        "your_client_id",
        authority="https://login.microsoftonline.com/common")
    ```

    >[!NOTE]
    > The authority is set to `/common` to allow sign ins with both organizaiton and personal Microsoft accounts. You can change it to `/organizations` to only allow sign ins with work and school accounts, `/consumers` to only allow personal Microsoft accounts, or with `/YOUR_TENANT_ID` to only allow sign ins from work and school accounts associated with your tenant.

    Instantiate a variable to hold the authentication result:

    ```python
    result = None  # It is just an initial value. Please follow instructions below.
    ```

2. Try and obtain the tokens from the cache first. The API model in MSAL provides you explicit control on how to utilize the token cache. While the caching part is technically optional, we highly recommend you to use it in your application. Using the cache you can ensure that you're not making any extra API calls and handle the token refresh automatically.

   ```python
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

3. If there is no suitable token in the cache or you've chosen to skip the previous step, send a request to Microsoft Entra ID to get a token. There are different methods based on your client type and scenario, but for the purposes of the example we're showing how to use [`acquire_token_interactive`](xref:msal.application.PublicClientApplication.acquire_token_interactive) which will prompt the user to provide their credentials.

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

4. Save the code into a Python file locally, such as `msaltest.py`.
5. Run the code by executing `python .\msalpytest.py`.

>[!NOTE]
> You can also download runnable samples from the [library repository](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.22.0/sample/interactive_sample.py).

If the application was configured correctly, you should see a web browser window appear asking the user to sign in.

![Example of an app prompting the user to sign in with their account](./media/basic-pca-app-prompt.gif)

Once the authentication is completed and you closed the browser, you should be able to see the access token printed in the terminal.

## Usage scenarios

MSAL Python can be used by applications to acquire tokens to access protected APIs. Tokens can be acquired by different **application types**: desktop applications, web applications, web APIs, and applications running on devices that don't have a browser (such as IoT devices). In MSAL Python, applications are categorized as follows:

- **Public client applications (desktop and mobile).** These types of apps cannot store app secrets securely.
- **Confidential client applications (web apps, web APIs, and daemon applications).** These type of apps securely store a secret registered with Microsoft Entra ID.

Learn more about instantiating and configuring the above in the [Client applications](./getting-started/client-applications.md) topic.

MSAL Python supports acquiring tokens either in the name of a user or in the name of the application itself (without a user). In the latter case, a confidential client application must be used.

MSAL Python can be used in applications running on different operating systems (Windows, Linux, macOS).

Key scenarios supported by MSAL Python:

- [Web application that signs in users](/azure/active-directory/develop/scenario-web-app-sign-user-overview)
- [Web Application signing in a user and calling a Web API in the name of the user](/azure/active-directory/develop/scenario-web-app-call-api-overview) (note that MSAL only helps the web application to sign in and obtain tokens. To [protect a web API](/azure/active-directory/develop/scenario-protected-web-api-overview), you will need other libraries).
- [Desktop application calling a Web API in the name of the signed-in user](/azure/active-directory/develop/scenario-desktop-overview)
- [Desktop/service daemon application calling Web API without a user](/azure/active-directory/develop/scenario-daemon-overview)
- [Application without a browser, or IoT application calling an API in the name of the user](/azure/active-directory/develop/scenario-desktop-acquire-token?tabs=python#command-line-tool-without-web-browser)

Can't find the scenario you are looking for? Check out the [supported scenarios and platforms](/azure/active-directory/develop/authentication-flows-app-scenarios#scenarios-and-supported-platforms-and-languages) across MSAL libraries.

## Releases

Refer to [MSAL Python releases on GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-python/releases).
