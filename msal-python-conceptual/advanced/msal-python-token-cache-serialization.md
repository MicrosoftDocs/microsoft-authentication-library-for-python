---
title: Custom token cache serialization (MSAL Python)
description: Learn how to serialize token cache using MSAL for Python
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: article
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
#Customer intent: 
---

# Custom token cache serialization in MSAL for Python

In Microsoft Authentication Library (MSAL) for Python, an in-memory token cache that persists for the duration of the app session, is provided by default when you create an instance of <xref:msal.application.ConfidentialClientApplication>.

Serialization of the token cache, so that different sessions of your app can access it, isn't provided "out of the box." MSAL for Python can be used in app types that don't have access to the file system--such as Web apps. To have a persistent token cache in an app that uses MSAL for Python, you must provide custom token cache serialization.

The strategies for serializing the token cache differ depending on whether you're writing a public client application (Desktop), or a confidential client application (web app, web API, or daemon app).

## Token cache for a public client application

Public client applications run on a user's device and manage tokens for a single user. In this case, you could serialize the entire cache into a file. Remember to provide file locking if your app, or another app, can access the cache concurrently. For a simple example of how to serialize a token cache to a file without locking, see the example in the <xref:msal.token_cache.SerializableTokenCache> class reference documentation.

## Token cache for a Web app (confidential client application)

For web apps or web APIs, you might use the session, or a Redis cache, or a database to store the token cache. There should be one token cache per user (per account) so ensure that you serialize the token cache per account.

## Next steps

See [ms-identity-python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/0.3.0/app.py#L66-L74) for an example of how to use the token cache for a Windows or Linux Web app or web API. The example is for a web app that calls the Microsoft Graph API.
