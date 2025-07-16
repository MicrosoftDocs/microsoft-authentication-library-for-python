---
title: Azure AD FS support (MSAL Python)
description: Learn about Active Directory Federation Services (AD FS) support in the Microsoft Authentication Library for Python
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: how-to
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
#Customer intent: 
---

# Active Directory Federation Services support in MSAL for Python

Active Directory Federation Services (AD FS) in Windows Server enables you to add OpenID Connect and OAuth 2.0 based authentication and authorization to your apps by using the Microsoft Authentication Library (MSAL) for Python. Using the MSAL for Python library, your app can authenticate users directly against AD FS. For more information about scenarios, see [AD FS Scenarios for Developers](/windows-server/identity/ad-fs/ad-fs-development).

There are usually two ways of authenticating against AD FS:

- MSAL Python talks to Microsoft Entra ID, which itself is federated with other identity providers. The federation happens through AD FS. MSAL Python connects to Microsoft Entra ID, which signs in users that are managed in Microsoft Entra ID (managed users) or users managed by another identity provider such as AD FS (federated users). MSAL Python doesn't  know that a user is federated. It simply talks to Microsoft Entra ID. The [authority](/azure/active-directory/develop/msal-client-application-configuration#authority) you use in this case is the usual authority (authority host name + tenant, common, or organizations).
- MSAL Python talks directly to an AD FS authority. This is only supported by AD FS 2019 and later.

## Connect to Active Directory federated with AD FS

### Acquire a token interactively for a federated user

The following applies whether you connect directly to Active Directory Federation Services (AD FS) or through Active Directory.

When you call `acquire_token_by_authorization_code` or `acquire_token_by_device_flow`, the user experience is typically as follows:

1. The user enters their account ID.
2. Microsoft Entra ID displays briefly the message "Taking you to your organization's page" and the user is redirected to the sign-in page of the identity provider. The sign-in page is usually customized with the logo of the organization.

The supported AD FS versions in this federated scenario are:
- Active Directory Federation Services FS v2
- Active Directory Federation Services v3 (Windows Server 2012 R2)
- Active Directory Federation Services v4 (AD FS 2016)

### Acquire a token via username and password

The following applies whether you connect directly to Active Directory Federation Services (AD FS) or through Active Directory.

When you acquire a token using `acquire_token_by_username_password`, MSAL Python gets the identity provider to contact based on the username. MSAL Python gets a [SAML 1.1 token](/azure/active-directory/develop/reference-saml-tokens) from the identity provider, which it then provides to Microsoft Entra which returns the JSON Web Token (JWT). We do not recommend the username and password flow as it presents security risks that are not present in other flows. For more information about why you want to avoid using this grant, see [why Microsoft is working to make passwords a thing of the past](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/). 


## Connecting directly to AD FS

When you connect directory to AD FS, the authority you'll want to use to build your application will be something like `https://somesite.contoso.com/adfs/`

MSAL Python supports ADFS 2019, but does not support a direct connection to ADFS 2016 or ADFS v2. Once you have upgraded your on-premises system to ADFS 2019, you can use MSAL Python.
