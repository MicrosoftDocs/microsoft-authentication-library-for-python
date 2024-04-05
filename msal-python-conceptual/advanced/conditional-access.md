---
title: Conditional access and claims challenges
description: When getting tokens silently, your application might receive errors when a Conditional Access claims challenge, such as MFA polic, is required by an API you're trying to access.
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
---

# Conditional access and claims challenges

## Background

When getting tokens silently, your application might receive errors when a [Conditional Access claims challenge](/azure/active-directory/develop/conditional-access-dev-guide), such as MFA policy, is required by an API you're trying to access.

The pattern for handling this error is to interactively acquire a token using MSAL. Interactively acquiring a token prompts the user and gives them the opportunity to satisfy the required Conditional Access policy.

> [!NOTE]
> There are certain conditional access policies that do not require user interaction to comply with the policy. In such cases, it is recommended to use the acquire_token_silent method to handle the claims challenge that is returned and avoid prompting user interaction.

In certain cases when calling an API requiring Conditional Access, you can receive a claims challenge in the error from the API. For instance if the Conditional Access policy is to have a managed device (Intune) the error will be something like
[AADSTS53000: Your device is required to be managed to access this resource](/azure/active-directory/develop/reference-aadsts-error-codes)
or something similar. In this case, you can pass the claims in the acquire token call so that the user is prompted to satisfy the appropriate policy.

## Handling Claim Challenge in MSAL Python

When calling an API requiring Conditional Access, your application will need to handle claim challenge error.
This will appear as
[an MSAL Python error response](/azure/active-directory/develop/msal-handling-exceptions?tabs=python)
where the Claims property won't be empty.

To handle the claim challenge, you'll need to use the `claims_challenge` parameter of the `get_authorization_request_url()` method. In cases that do not require interactions, you'll need to use the `claims_challenge` parameter in the `acquire_token_silent()` method and the method will use the refresh token to obtain a new token.

**Claims challenge** is a directive in the *www-authenticate* header that the resource provider(ex: MS Graph) will respond with, when an access token is not authorized and a new access token is required. An example claims challenge from resource provider(ex: MS Graph) should look like this. It consists of HTTP Status code which must be 401 and *www-authenticate* header which contains the fields mentioned above.

```http
HTTP 401; Unauthorized 
www-authenticate=Bearer realm="", 
authorization_uri="https://login.microsoftonline.com/common/oauth2/authorize", 
error="insufficient_claims", 
claims="returned_claims"
```

Clients need to extract the claims and then pass it to acquire_token_by_* method. If the claims value returned from the resource is base64-encoded, it needs to be decoded before calling MSAL. MSAL acquire_token methods accept claims as a json string. This is an example of how claims returned from the resource provider can be handled using authorization code flow.

```python
claims_challenge = "claims returned from resource provider"
# if claims parameter is encoded, decode it and use the decoded claims
# decoded_claims_challenge = str(base64.b64decode(claims), "utf-8")

result = None

# Since you were using your previous AT and then met a claims challenge,
# you should've already be in a context with the current signed-in user.
# Here we assume you somehow already have that account,
# or you could possibly get it like this:
accounts = app.get_accounts()
assert accounts
account = accounts[0]

result = app.acquire_token_silent(scope, account=account, claims_challenge=claims_challenge)
if not result:
    # Auth endpoint call for interactive login (you can use auth url helper)
    auth_url = app.get_authorization_request_url(..., claims_challenge=claims_challenge)
    # Make a request to this auth_url to get back authorization_code
    result = app.acquire_token_by_authorization_code(..., claims_challenge=claims_challenge)
```

- This new access token can now be used in the request to resource.
