---
title: Username and password authentication
description: "By design and policy, the username/password authentication works only for Work and school accounts, but not for Microsoft Accounts (MSA)."
author: Dickson-Mwendia
manager: CelesteDG
ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
ms.custom: sfi-image-nochange
#Customer intent: 
---

# Username and password authentication

The content below are applicable to [all MSAL libraries](/entra/msal), not just MSAL Python.

## The username and password flow is not recommended

Microsoft recommends you do not use the username and password flow. In most scenarios, more secure alternatives are available and recommended. This flow requires a very high degree of trust in the application, and carries risks that are not present in other flows. You should only use this flow when other more secure flows aren't viable. For more information about why you want to avoid using this grant, see [why Microsoft is working to make passwords a thing of the past](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/).

## Constraints

* By design and policy, the username/password authentication works only for Work and school accounts, but not for Microsoft Accounts (MSA).
  See the [definition of these 2 types of accounts here](/azure/active-directory/fundamentals/sign-up-organization).
* The Username/Password authentication is not compatible with conditional access and multi-factor authentication,
  because this is not an interactive flow, the Microsoft identity platform does not have an opportunity to present a web-based dialog for the end user to interact.
  As a consequence, if your app runs in a Microsoft Entra tenant where the tenant admin requires multi-factor authentication (many organizations do that), this flow will not work.
* Because Username Password Authentication is a non-interactive flow:
  - the user of your application must have previously consented to use the application 
  - or the tenant admin must have previously consented to all users in the tenant to use the application.
  - This means that:
     - either you as a developer have pressed the **Grant** button on the Azure portal for yourself, 
     - or a tenant admin has pressed the **Grant/revoke admin consent for {tenant domain}** button in the **API permissions** tab of the registration for the application (See [Add permissions to access web APIs](/azure/active-directory/develop/quickstart-configure-app-access-web-apis#add-permissions-to-access-web-apis))
     - or you have provided a way for users to consent to the application (See [Requesting individual user consent](/azure/active-directory/develop/v2-permissions-and-consent#requesting-individual-user-consent))
     - or you have provided a way for the tenant admin to consent for the application (See [admin consent](/azure/active-directory/develop/v2-permissions-and-consent#requesting-consent-for-an-entire-tenant))

## Recommendations

Even if you choose to use Username Password Authentication, you should not persist end user's password. After the initial username password authentication succeeded, MSAL's token cache would kick in, and cache the refresh token (RT) automatically. From now on, your app can just call MSAL's [`acquire_token_silent()`](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.acquire_token_silent) to obtain new access token without username and password.

## Setup

Microsoft identity platform supports username password flow on public client application and Confidential client application.
If you would need to configure your app as a public client application, turn on the switch on below screenshot to allow it.

![Public App Setup](https://user-images.githubusercontent.com/821550/76988648-4499c280-6902-11ea-8be5-00292624a274.png)
