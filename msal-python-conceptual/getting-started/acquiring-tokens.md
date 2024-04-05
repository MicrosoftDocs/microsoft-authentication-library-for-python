---
title: Acquire tokens for your app
description: Learn how to acquire tokens for your Python appliccation. You can acquire tokens silently or interactively through a web browser.
author: Dickson-Mwendia
manager: CelesteDG
ms.service: msal
ms.subservice: msal-python
ms.topic: conceptual
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
zone_pivot_groups: msal-python-acquire-token
zone_pivot_group_filename: msal/python/zone-pivot-groups.json
---

# Acquiring tokens

As explained in [MSAL Python usage scenarios](../index.md#usage-scenarios), there are many ways of acquiring a token. Some require user interaction through a web browser, while others don't require any user interaction at all.

## Prerequisites

Before acquiring tokens with MSAL Python, make sure to instantiate a [client application](./client-applications.md)

## Token acquisition methods

The approach to acquiring a token is different depending on the application type - public client applications (desktop and mobile) or confidential client application (web app, web API, or a daemon application like a Windows service). Each of the individual approaches is described below.

:::zone pivot="pca"

### Public client applications

Public client applications cannot securely store a secret and can *only* authenticate the user that is interacting with the product. MSAL Python exposes most of the token acquisition logic for public applications through [`PublicClientApplication`](xref:msal.application.PublicClientApplication). Using this class enables developers to:

- Acquire tokens **as the user** with the help of an [interactive flow](/azure/active-directory/develop/scenario-desktop-acquire-token?tabs=python#acquiring-a-token-interactively) after letting the user sign-in through the authorization request URL.
- It's also possible (but not recommended) to get a token with a [username and password](/azure/active-directory/develop/scenario-desktop-acquire-token?tabs=python#username--password).
- For applications running on devices which don't have a web browser, it's possible to acquire a token through the [device code flow](/azure/active-directory/develop/scenario-desktop-acquire-token?tabs=python#command-line-tool-without-web-browser), which provides the user with a URL and a code. The user goes to a web browser on another device, enters the code and signs in. On successful authentication, Microsoft Entra ID will return a token to the browser-less device.

:::zone-end

:::zone pivot="cca"

### Confidential client applications

Confidential client applications can securely store a secret and can authenticate both on behalf of an application as well as on behalf of a given user. With MSAL Python, developers can use [`ConfidentialClientApplication`](xref:msal.application.ConfidentialClientApplication) to access confidential client application capabilities, such as:

- Acquire token **as the application itself** using [client credentials](/azure/active-directory/develop/scenario-daemon-acquire-token?tabs=python#acquiretokenforclient-api), and not for a user. For example, this can be used in applications which process users in batches and not one particular user, such as syncing tools.
- In the case of web Apps or web APIs **calling another downstream Web API in the name of the user**, use the [On Behalf Of flow](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to acquire a token based on a user assertion (such as Security Assertion Markup Language (SAML), JWT).
- **For Web apps authenticating in the name of a user**, acquire tokens through [authorization code](/azure/active-directory/develop/scenario-web-app-call-api-acquire-token?tabs=python) after letting the user sign-in through the authorization request URL. This is typically the mechanism used by an application which lets the user sign-in using OpenID Connect but then wants to access Web APIs for this particular user.

:::zone-end

## MSAL Python token caching

Both public and confidential client applications support token caching, handled direclt by MSAL Python. Applications should try to get a token from the cache first before relying on any other means. Take a look at the [recommended token acquisition pattern](/azure/active-directory/develop/scenario-desktop-acquire-token?tabs=python) to learn more.

To be able to persist the cache, the developers will need to configure the [token cache serialization](/azure/active-directory/develop/msal-python-token-cache-serialization) logic.
