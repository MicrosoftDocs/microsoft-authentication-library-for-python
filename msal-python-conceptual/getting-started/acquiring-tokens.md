---
title: Acquire tokens for your app
description: Learn how to acquire tokens for your Python application. You can acquire tokens silently or interactively through a web browser.
author: SHERMANOUKO
manager: CelesteDG
ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 04/24/2025
ms.author: shermanouko
ms.reviewer: dmwendia, rayluo
# zone_pivot_groups: msal-python-acquire-token
# zone_pivot_group_filename: msal/python/zone-pivot-groups.json
#Customer intent: 
---

# Acquire tokens

There are many ways of acquiring a token with MSAL Python. Some require user interaction while others don't. The approach used to acquire a token is different depending on whether the developer is building a public client (desktop or mobile) or a confidential client application (web app, web API, or daemon like a Windows service).

## Prerequisites

Before you acquire tokens with MSAL Python, learn about [types of client application](./client-applications.md).

## Get user account

An app can acquire a token as itself or on behalf of a user. To acquire a token on behalf of a user, the app needs to know the user's account. MSAL Python provides the [`get_accounts`](/python/api/msal/msal.application.clientapplication#msal-application-clientapplication-get-accounts) method to get the user's account. This method is available on both [`PublicClientApplication`](/python/api/msal/msal.application.publicclientapplication) and [`ConfidentialClientApplication`](/python/api/msal/msal.application.confidentialclientapplication) classes. The method returns a list of accounts that the user has previously signed in with, that is, exists in cache.

```python
accounts = app.get_accounts(username=user.get("preferred_username"))
```

The account selected by the user for sign-in can later be used in `acquire_token_silent()` to find its tokens.

## Token grant flows

There are several authentication flows that can be used to acquire tokens with MSAL Python. You can find more information on these flows on the [Microsoft identity platform documentation](/entra/identity-platform/v2-oauth2-auth-code-flow).

> [!WARNING]
> Always use MSAL to get security tokens and call protected web APIs in your apps. We don't recommend you implementing your own token acquisition logic. These flows are to help you have a better understanding of how things work. If you're securing a web application, we recommend using the [identity](https://pypi.org/project/identity/) library. This library isn't officially maintained by Microsoft but it implements most of the logic you need to acquire tokens in web apps.

## Interactive vs silent

MSAL Python supports both interactive and silent token acquisition. Interactive token acquisition requires user interaction, while silent token acquisition doesn't. Public clients generally require user interaction while confidential clients rely on pre-provisioned credentials, like certificates and secrets.

Use the [`acquire_token_silent_with_error`](/python/api/msal/msal.application.clientapplication#msal-application-clientapplication-acquire-token-silent-with-error) method to silently acquire a token. This method finds a valid access token from cache, or a valid refresh token from cache and then automatically uses it to redeem a new access token. If neither is true, you need to use an interactive method to acquire the token.

If your app doesn't care about the exact token refresh error during token cache look-up, then the [`acquire_token_silent`](/python/api/msal/msal.application.clientapplication#msal-application-clientapplication-acquire-token-silent) method is recommended.

An example usage of this method is as shown in the following code snippet.

```python
if accounts:
    # If so, you could then somehow display these accounts and let end user choose
    chosen = accounts[0]
    result = app.acquire_token_silent(scopes=["your_scope"], account=chosen)
    
    # At this point, you can save you can update your cache if you are using token caching
    # check result variable, if its None then you should interactively acquire a token
    if not result:
        # So no suitable token exists in cache. Let's get a new one from Microsoft Entra.
        result = app.acquire_token_by_one_of_the_actual_method(..., scopes=["User.Read"])
    
    if "access_token" in result:
        access_token = result["access_token"]
    else:
        print(result.get("error"))  
        print(result.get("error_description"))
        print(result.get("correlation_id"))  # You may need this when reporting a bug
```

Several methods are available for interactive token acquisition. The method to use depends on the type of app you're building and the token grant flow applicable to your scenario.

## Public clients interactive token acquisition

Public client applications can't securely store a secret and can only authenticate the user that is interacting with the product. MSAL Python exposes the token acquisition logic for public applications through [`PublicClientApplication`](xref:msal.application.PublicClientApplication). The following are the different methods available for public client applications to acquire tokens.

### Device code flow

[Device code flow](/entra/identity-platform/v2-oauth2-device-code) is used to acquire tokens in applications that run on devices that don't have access to a web browser. These are applications are known as headless applications. This flow provides the user with a URL and a code. The user goes to a web browser on another device, enters the code and signs in. On successful authentication, Microsoft Entra returns a token to the browser-less device.

First, you call the [`initiate_device_flow`](/python/api/msal/msal.application.publicclientapplication#msal-application-publicclientapplication-initiate-device-flow) method.

```python
flow = app.initiate_device_flow(scopes=config["scope"])
if "user_code" not in flow:
    raise ValueError(
        "Fail to create device flow. Err: %s" % json.dumps(flow, indent=4))

print(flow["message"])
sys.stdout.flush()  # Some terminal needs this to ensure the message is shown

# Ideally you should wait here, in order to save some unnecessary polling
# input("Press Enter after signing in from another device to proceed, CTRL+C to abort.")
```

You then pass the flow dictionary object to the [`acquire_token_by_device_flow`](/python/api/msal/msal.application.publicclientapplication#msal-application-publicclientapplication-acquire-token-by-device-flow) method to get the token. By default, this method blocks the current thread. You can follow [these instructions](/python/api/msal/msal.application.publicclientapplication#msal-application-publicclientapplication-acquire-token-by-device-flow) to shorten the block time or you can even turn off the blocking behavior and then keep calling `acquire_token_by_device_flow` in your own customized loop.

```python
result = app.acquire_token_by_device_flow(flow)

if "access_token" in result:
    access_token = result["access_token"]
else:
    print(result.get("error"))  
```

A successful response a dictionary with an `access_token` key.

### Acquire token interactive

MSAL Python also offers the ability for public client apps (Desktops and Mobile) to acquire tokens as the user. The user signs in through the authorization request URL via a web browser. Set the redirect URI of your app to `http://localhost` in the Microsoft Entra admin center for your app registration. If you opt in to [use broker](../advanced/wam.md) during `PublicClientApplication` creation, your app also needs to register `ms-appx-web://Microsoft.AAD.BrokerPlugin/YOUR_CLIENT_ID` as a redirect URI.

```python
result = app.acquire_token_interactive(  # It automatically provides PKCE protection
    scopes=config["scope"])

if "access_token" in result:
    access_token = result["access_token"]
else:
    print(result.get("error"))  
```

### Username and password

We don't recommend using this approach. It's also possible to get a token with a [username and password](/entra/identity-platform/v2-oauth-ropc). MSAL Python provides the [`acquire_token_by_username_password`](/python/api/msal/msal.application.clientapplication#msal-application-clientapplication-acquire-token-by-username-password) method for this use case. It's not recommended because the application will be asking a user for their password directly, which is an insecure pattern.

Microsoft doesn't recommend the username and password flow because the application will be asking a user for their password directly, which is an insecure pattern. In most scenarios, there exist more secure flows that you can use. Learn more in the [username and password authentication flow](../advanced/username-password-authentication.md) guidance.

```python
result = app.acquire_token_by_username_password(
    username=config["username"], password=config["password"], scopes=config["scope"])

if "access_token" in result:
    access_token = result["access_token"]
else:
    print(result.get("error"))  
```

## Confidential clients interactive token acquisition

Confidential client applications can securely store a secret and can authenticate both on behalf of an application as well as on behalf of a given user. MSAL Python gives developers various methods to acquire tokens when developing [`ConfidentialClientApplication`](xref:msal.application.ConfidentialClientApplication).

### Acquire token for client

Acquire token as the application itself using [client credentials](/entra/identity-platform/v2-oauth2-client-creds-grant-flow), and not for a user. For example, this can be used in applications that process users in batches and not one particular user, such as syncing tools. MSAL Python provides the [`acquire_token_for_client`](/python/api/msal/msal.application.confidentialclientapplication#msal-application-confidentialclientapplication-acquire-token-for-client) method to do this. Since MSAL Python 1.23, this method automatically looks for token from cache, and only sends request to identity provider when cache misses.

```python
result = app.acquire_token_for_client(scopes=config["scope"])

if "access_token" in result:
    access_token = result["access_token"]
else:
    print(result.get("error"))    
```

### Acquire token on behalf of

In the case of web apps or web APIs calling another downstream Web API in the name of the user, use the [On Behalf Of flow](/entra/identity-platform/v2-oauth2-on-behalf-of-flow) to acquire a token based on a user assertion. For example, SAML and JWT. The current app is a middle-tier service that was called with a token representing an end user. The current app can use such token, also known as a user assertion, to request another token to access downstream web API on behalf of that user. The middle-tier app has no user interaction to obtain consent. For information on gaining consent upfront for your middle-tier app, see the [documentation](/entra/identity-platform/v2-oauth2-on-behalf-of-flow#gaining-consent-for-the-middle-tier-application).

Here's an example of code that acquires an access token using the [`acquire_token_on_behalf_of`](/python/api/msal/msal.application.confidentialclientapplication#msal-application-confidentialclientapplication-acquire-token-on-behalf-of) method.

```python
def get(self, request): # a web service endpoint receiving a request
    
    scopes = ["your-scopes"]
    downstream_api = "https://your-downstreamapi.com/resource" #your downstream API resource endpoint
    current_access_token = request.headers.get("Authorization", None)
    
    # initialize the app
    app = msal.ConfidentialClientApplication(...) # refer to initialization of the app documentation

    #acquire token on behalf of the user that called this API
    downstream_api_access_token = app.acquire_token_on_behalf_of(
        user_assertion=current_app_access_token.split(' ')[1],
        scopes=_scopes
    )

    if "access_token" in result:
        access_token = result["access_token"]
        # use access_token to call dowstream API e.g
        requests.get(downstream_api, headers={'Authorization': f'Bearer {downstream_api_access_token}'})
    else:
        print(result.get("error")) 
```

### Acquire token by authorization code flow

For web apps authenticating in the name of a user, acquire tokens through [authorization code](/entra/identity-platform/v2-oauth2-auth-code-flow) after letting the user sign-in through the authorization request URL. This is typically the mechanism used by an application that lets the user sign-in and access web APIs for this particular user.

You'll first need to initiate auth code flow using the [`initiate_auth_code_flow`](/python/api/msal/msal.application.clientapplication#msal-application-clientapplication-initiate-auth-code-flow). This method takes in among other parameters a redirect URI and state string. The value of the state parameter is also included in token response. If this value is absent, MSAL Python will automatically generate one internally. The redirect URI provided must match the redirect URI registered in the Microsoft Entra admin center. This method returns the auth code flow that is a dictionary containing `auth_uri` and `state`. The `auth_uri` is the URL that the user needs to visit to sign in.

```python
flow = app.initiate_auth_code_flow(
    scopes=config["scope"], redirect_uri=config["redirect_uri"], state="your-state-value")

if "error" in flow:
    print(flow.get("error"))

# Save the response somewhere e.g in session
session["auth_flow"] = flow

# At this point, the app should guide the user to visit the auth ur (session["auth_flow"]["auth_uri"])
```

The response from visiting the auth uri endpoints is used in the [`acquire_token_by_auth_code_flow`](/python/api/msal/msal.application.clientapplication#msal-application-clientapplication-acquire-token-by-auth-code-flow) method. The state is a unique identifier that you can use to verify the response from the authorization server. The user should consent to scopes during sign-in.

```python
# The uth_response value from visiting the auth_uri endpoint is passed as a query string
# You can change this by passing a value to the response_mode in the initiate_auth_code_flow method
try:
    result = app.acquire_token_by_auth_code_flow(session.get("flow", {}), auth_response)
    
    if "access_token" in result:
        access_token = result["access_token"]
    else:
        print(result.get("error"))
except ValueError:  # Usually caused by CSRF
    pass  # Simply ignore them
```

## MSAL Python token caching

Both public and confidential client applications support token caching, handled direct by MSAL Python. Applications should try to get a token from the cache first before relying on any other means. For more information, see [recommended token acquisition pattern](/azure/active-directory/develop/scenario-desktop-acquire-token?tabs=python).

To be able to persist the cache, the developers need to configure the [token cache serialization](/azure/active-directory/develop/msal-python-token-cache-serialization) logic.
