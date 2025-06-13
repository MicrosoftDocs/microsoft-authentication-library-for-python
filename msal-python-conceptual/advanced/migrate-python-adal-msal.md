---
title: Python ADAL to MSAL migration guide
description: Learn how to migrate your Azure Active Directory Authentication Library (ADAL) Python app to the Microsoft Authentication Library (MSAL) for Python.
author: Dickson-Mwendia
manager: CelesteDG
ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
ms.custom: sfi-ropc-nochange
#Customer intent: 
---

# ADAL to MSAL migration guide for Python

This article highlights changes you need to make to migrate an app that uses the Azure Active Directory Authentication Library (ADAL) to use the Microsoft Authentication Library (MSAL).

You can learn more about MSAL and get started with an [overview of the Microsoft Authentication Library for Python](../index.md).

## Difference highlights

ADAL works with the Azure Active Directory (Azure AD) v1.0 endpoint. The Microsoft Authentication Library (MSAL) works with the Microsoft identity platform--formerly known as the Azure Active Directory v2.0 endpoint. The Microsoft identity platform differs from Azure AD v1.0 in that it:

Supports:

- Work and school accounts (Microsoft Entra ID provisioned accounts)
- Personal accounts (such as Outlook.com or Hotmail.com)
- Your customers who bring their own email or social identity (such as LinkedIn, Facebook, Google) via the Azure AD B2C offering

- Is standards compatible with:
  - OAuth v2.0
  - OpenID Connect (OIDC)

For more information about MSAL, see [MSAL overview](../index.md).

### Scopes not resources

ADAL Python acquires tokens for resources, but MSAL Python acquires tokens for scopes. The API surface in MSAL Python doesn't have resource parameter anymore. You would need to provide scopes as a list of strings that declare the desired permissions and resources that are requested. To see some example of scopes, see [Microsoft Graph's scopes](/graph/permissions-reference).

You can add the `/.default` scope suffix to the resource to help migrate your apps from the v1.0 endpoint (ADAL) to the Microsoft identity platform (MSAL). For example, for the resource value of `https://graph.microsoft.com`, the equivalent scope value is `https://graph.microsoft.com/.default`. If the resource isn't in the URL form, but a resource ID of the form `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX`, you can still use the scope value as `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX/.default`.

For more details about the different types of scopes, refer to [Permissions and consent in the Microsoft identity platform](/azure/active-directory/develop/permissions-consent-overview) and the [Scopes for a Web API accepting v1.0 tokens](/azure/active-directory/develop/msal-v1-app-scopes) articles.

### Error handling

ADAL for Python uses the exception `AdalError` to indicate that there's been a problem. MSAL for Python typically uses error codes, instead. For more information, see [MSAL for Python error handling](msal-error-handling-python.md).

### API changes

The following table lists an API in ADAL for Python, and the one to use in its place in MSAL for Python:

| ADAL for Python API | MSAL for Python API |
| ------------------- | ------------------- |
| [AuthenticationContext](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext)                                                                                                                                                                                                                       | <xref:msal.application.PublicClientApplication> or <xref:msal.application.ConfidentialClientApplication>                                                                                                                                                                                |
| N/A                                                                                                                                                                                                                                                                                                                     | <xref:msal.application.PublicClientApplication.acquire_token_interactive>                                                                                                                                                                                                                                                                                                                     |
| N/A                                                                                                                                                                                                                                                                                                                     | <xref:msal.application.ClientApplication.get_authorization_request_url>                                                                                                                                                                                                                                                                                                            |
| N/A                                                                                                                                                                                                                                                                                                                     | <xref:msal.application.ClientApplication.initiate_auth_code_flow>                                                                                                                                                                                                                                                                                                            |
| [acquire_token_with_authorization_code()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_authorization_code)                                                                                                                                                               | <xref:msal.application.ClientApplication.acquire_token_by_auth_code_flow>                                                                                                                                                                                                                                                                                             |
| [acquire_token()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token)                                                                                                                                                                                                               | <xref:msal.application.ClientApplication.acquire_token_silent>                                                                                                                                                          |
| [acquire_token_with_refresh_token()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_refresh_token)                                                                                                                                                                         | These two helpers are intended to be used during [migration](#migrate-existing-refresh-tokens-for-msal-python) only: <xref:msal.application.ClientApplication.acquire_token_by_refresh_token> |
| [acquire_user_code()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_user_code)                                                                                                                                                                                                       | <xref:msal.application.PublicClientApplication.initiate_device_flow>                                                                                                                                                                                                                                                                                                                                                       |
| [acquire_token_with_device_code()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_device_code) and [cancel_request_to_get_token_with_device_code()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.cancel_request_to_get_token_with_device_code) | <xref:msal.application.PublicClientApplication.acquire_token_by_device_flow>                                                                                                                                                                                                                                                                                                                                       |
| [acquire_token_with_username_password()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_username_password)                                                                                                                                                                 | <xref:msal.application.ClientApplication.acquire_token_by_username_password>                                                                                                                                                                                                                                                                                                                           |
| [acquire_token_with_client_credentials()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_client_credentials) and [acquire_token_with_client_certificate()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_client_certificate) | <xref:msal.application.ConfidentialClientApplication.acquire_token_for_client>                                                                                                                                                                                                                                                                                        |
| N/A                                                                                                                                                                                                                                                                                                                     | <xref:msal.application.ConfidentialClientApplication.acquire_token_on_behalf_of>                                                                                                                                                                                                                                                                                                                     |
| [TokenCache()](https://adal-python.readthedocs.io/en/latest/#adal.TokenCache)                                                                                                                                                                                                                                           | <xref:msal.token_cache.SerializableTokenCache>                                                                                                                                                                                                                                                                                                                                                                           |
| N/A                                                                                                                                                                                                                                                                                                                     | Cache with persistence, available from [MSAL Extensions](https://github.com/marstr/original-microsoft-authentication-extensions-for-python)                                                                                                                                                                                                                                                                                                                                     |

## Migrate existing refresh tokens for MSAL Python

MSAL abstracts the concept of refresh tokens. MSAL Python provides an in-memory token cache by default so that you don't need to store, lookup, or update refresh tokens. Users will also see fewer sign-in prompts because refresh tokens can usually be updated without user intervention. For more information about the token cache, see [Custom token cache serialization in MSAL for Python](msal-python-token-cache-serialization.md).

The following code will help you migrate your refresh tokens managed by another OAuth2 library (including but not limited to ADAL Python) to be managed by MSAL for Python. One reason for migrating those refresh tokens is to prevent existing users from needing to sign in again when you migrate your app to MSAL for Python.

The method for migrating a refresh token is to use MSAL for Python to acquire a new access token using the previous refresh token. When the new refresh token is returned, MSAL for Python will store it in the cache.
Since MSAL Python 1.3.0, we provide an API inside MSAL for this purpose.
Please refer to the following code snippet, quoted from
[a completed sample of migrating refresh tokens with MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.3.0/sample/migrate_rt.py#L28-L67)

```python
import msal
def get_preexisting_rt_and_their_scopes_from_elsewhere():
    # Maybe you have an ADAL-powered app like this
    #   https://github.com/AzureAD/azure-activedirectory-library-for-python/blob/1.2.3/sample/device_code_sample.py#L72
    # which uses a resource rather than a scope,
    # you need to convert your v1 resource into v2 scopes
    # See https://learn.microsoft.com/azure/active-directory/develop/migrate-python-adal-msal#scopes-not-resources
    # You may be able to append "/.default" to your v1 resource to form a scope
    # See https://learn.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent#the-default-scope

    # Or maybe you have an app already talking to the Microsoft identity platform,
    # powered by some 3rd-party auth library, and persist its tokens somehow.

    # Either way, you need to extract RTs from there, and return them like this.
    return [
        ("old_rt_1", ["scope1", "scope2"]),
        ("old_rt_2", ["scope3", "scope4"]),
        ]


# We will migrate all the old RTs into a new app powered by MSAL
app = msal.PublicClientApplication(
    "client_id", authority="...",
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.readthedocs.io/en/latest/#msal.SerializableTokenCache
    )

# We choose a migration strategy of migrating all RTs in one loop
for old_rt, scopes in get_preexisting_rt_and_their_scopes_from_elsewhere():
    result = app.acquire_token_by_refresh_token(old_rt, scopes)
    if "error" in result:
        print("Discarding unsuccessful RT. Error: ", json.dumps(result, indent=2))

print("Migration completed")
```
