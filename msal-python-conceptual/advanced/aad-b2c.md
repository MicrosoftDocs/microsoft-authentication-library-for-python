---
title: Use MSAL Python to work with Azure AD B2C
description: "You can use MSAL Python to sign-in users with social identities, acquire tokens, and customize the sign-in experience by using Azure AD B2C."
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
#Customer intent: 
---

# Use MSAL Python to work with Azure AD B2C

You can use MSAL Python to sign-in users with social identities, acquire tokens, and customize the sign-in experience by using [Azure AD B2C](https://aka.ms/aadb2c).

> [!IMPORTANT]
> Effective May 1, 2025, Azure AD B2C will no longer be available to purchase for new customers. To learn more, please see [Is Azure AD B2C still available to purchase?](/azure/active-directory-b2c/faq?tabs=app-reg-ga#azure-ad-b2c-end-of-sale) in our FAQ.

Azure AD B2C is built around the notion of [User Flows](/azure/active-directory-b2c/active-directory-b2c-reference-policies) (formerly known as policies). In MSAL Python, specifying a user flow translates to providing an authority.

* When you instantiate the client application, you need to specify the user flow in authority as
  `https://{tenant_name}.b2clogin.com/{tenant_name}.onmicrosoft.com/{user_flow}`.
  And then every usual `acquire_token_by_xyz(...)` would work with that user flow.
  There is no API surface difference between B2C and non-B2C scenarios.
  Just swap a different set of configuration, and the same
  [web app sample](https://github.com/Azure-Samples/ms-identity-python-webapp)
  would work for both non-B2C and B2C scenarios.
* Some user flows, such as Edit Profile user flow or Reset Password user flow,
  are essentially customized user experience. You can get a URL to those pages
  by creating a `PublicClientApplication` with the authority containing that `{user_flow}`,
  and then calling the `get_authorization_request_url(...)`.
  (In our [web app sample](https://github.com/Azure-Samples/ms-identity-python-webapp),
  we even provide a higher level helper for them.)

More details of these summary are described below.

## Authority for a B2C tenant and user flow

The authority to use is `https://{tenant_name}.b2clogin.com/{tenant_name}.onmicrosoft.com/{user_flow}` where:

- `tenant_name` is the name of the Azure AD B2C tenant, such as "contoso"
- `user_flow` is the name of the user flow to apply (for instance you might have "b2c_1_susi" for sign-in/sign-up).

When building the application, you need to provide, as usual, the authority, built as above

```python
app = msal.PublicClientApplication(  # Or ConfidentialClientApplication(...)
    "your_client_id",
    authority="https://contoso.b2clogin.com/contoso.onmicrosoft.com/b2c_1_susi",
    ...)
```

> [!NOTE]
> You may also use your own customized domain name, so that your authority would look like
> "https://**contoso.com**/{tenant_name}.onmicrosoft.com/{user_flow}".
> Under the hood, MSAL Python will automatically adjust the validate_authority check behavior for all B2C domains.
> So that you do not have to do anything extra.

## Acquiring a token

Acquiring a token for an Azure AD B2C protected API, based on the user flow already provided as part of the authority,
are exactly the same as what you would do in non-B2C scenario.
That's why our non-B2C [Python web app sample](https://github.com/Azure-Samples/ms-identity-python-webapp)
doubles as a B2C web app sample. Its major file `app.py` works on both non-B2C and B2C scenarios without any change.

```python
app.acquire_token_by_xyz(...)  # Same as in non-B2C scenarios
```

> There is no need to filter accounts by user flow,
as long as you are following a pattern of
"create different MSAL app for different user flow"
(because the B2C user flow is designed to behave like an isolated authority).
In practice, you will still typically reuse same MSAL app and its token cache for the SignIn user flow,
and only create new one-time MSAL app when invoking EditProfile or ResetPassword user flows,
whose returned token (if any) would not be useful anyway.

## Example of EditProfile and ResetPassword user flows

EditProfile and ResetPassword user flows are:

* less about acquiring tokens.
  That might not be obvious, but it is because currently the tokens issued by one B2C user flow
  are not necessarily compatible with tokens issued by another B2C user flow.
  Nonetheless, the safer pattern is to organize your web app to not share those tokens.
* more about a customized user experience.
  Imagine in an old school web app (even outside of the context of MSAL or B2C),
  how would you provide a customized Edit Profile experience?
  You would include a link into your web app's navigation bar, pointing to that customized page.

[Our B2C web app sample](https://github.com/Azure-Samples/ms-identity-python-webapp/commit/6b7cb85b79571f0164b3cb13ab998e3976293739#diff-fb5aa1cd1261d08d02db6f7dc314d9ab)
is exactly organized that way to hide all the underlying decisions.
You would just need to update your HTML template to include new link to, for example, the Edit Profile page.

```html
<a href='{{_build_auth_url(authority=config["B2C_PROFILE_AUTHORITY"])}}'>Edit Profile</a>
```

## Resource Owner Password Credentials (ROPC) With B2C

There exists no API difference between a B2C and non-B2C scenario. The following content serves as a mini-tutorial.

* In your Azure AD B2C tenant, create a new user flow and select **Sign in using ROPC**.
This will enable the ROPC user flow for your tenant.
See [Configure the resource owner password credentials flow](/azure/active-directory-b2c/configure-ropc) for more details.
* Once you create the MSAL instance with the authority which contains the ROPC user flow,
the [`acquire_token_by_username_password(...)`](https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.acquire_token_by_username_password)
would work as usual.
* Limitations: This **only works for local accounts** (where you register with B2C using an email or username). This flow does not work if federating to any of the IdPs supported by B2C (Facebook, Google, etc...).

Microsoft [discourages the use of resource owner password credentials grant](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/). In most scenarios, more secure alternatives are available and recommended. This flow requires a very high degree of trust in the application, and carries risks that are not present in other flows. You should only use this flow when other more secure flows aren't viable. Learn more in the [username and password ](username-password-authentication.md) guidance.

## Caching with B2C in MSAL Python

### Known issue

MSAL Python token cache usage pattern starts with querying all existing accounts by
[`get_accounts(...)`, which supports a `username` parameter as filter](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.get_accounts).
That username data is populated by a `preferred_username` claim inside the ID Token.

By default, that claim is missing in many of the Azure AD B2C scenarios. 

The customer impact is that when trying to display the accounts, their username field would be empty.
This may not bother you, if you are using Auth Code flow in your web app, and dealing with only one account per user.
But if you are using ROPC flow and feels you already know the end user's username,
`accounts = app.get_accounts(username="john.doe@contoso.com")` would still give you an empty result,
because `"john.doe@contoso.com"` won't match `""`.

The workaround is to customize your B2C user flow to populate and return a `preferred_username` claim,
or simply call your `app.get_accounts()` without a specific username parameter.

## Samples

Sample | Platform | Description
------ | -------- | -----------
[Microsoft Identity Python Web App](https://github.com/Azure-Samples/ms-identity-python-webapp) | All platforms supporting Python | A web app showcasing how to use MSAL Python to authenticate users via Azure Active Directory B2C, and access a Web API with the resulting tokens.
