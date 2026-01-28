---
title: Client capabilities
description: "The Microsoft Entra service provides features and policies which are applicable in certain scenarios, such as Conditional Access policies."
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-python
ms.topic: concept-article
ms.date: 02/07/2024
ms.author: dmwendia
ms.reviewer: shermanouko, rayluo
#Customer intent: 
---

# Client capabilities

The Microsoft Entra service provides features and policies which are applicable in certain scenarios, such as [Conditional Access](./conditional-access.md) policies. The Microsoft Entra service needs to determine if a client application is participating in the feature or capable of handling the policy, for the end to end scenario to work.

- The **client capabilities** parameter in the application object allows client applications to indicate their compliance with a scenario and readiness to handle the policies or features, so that Microsoft Entra ID can apply them for the client. 

    For example, in the context of Conditional Access scenario, adding client capability `CP1` will mean that the client is capable of handling a `claims challenge` from the resource provider(ex: MS Graph). Thus, Microsoft Entra STS (Security token service) will emit claims in the access token that may lead to claims challenge from a resource provider (ex: MS Graph).

- These capabilities are set when creating the application object using the `client_capabilities` parameter which is a list of client capability strings.

    ```python
    app = msal.PublicClientApplication(client_id="client_id", authority="your_authority", client_capabilities = ["CP1"])
    ```

- Once these are set at the application level, they will be sent in all the requests to the Microsoft Entra authorize and token endpoints. 
- When setting a capability, the client application should make sure that the policy or feature is handled in the application.
  - In the above example,  when such a capability(`CP1`) is set on the application object, the client application should make sure that a claims challenge from the resource provider(ex: MS Graph) is handled in the application. You can read more details about it in [handling the claims challenge ](./conditional-access.md#handling-claim-challenge-in-msal-python).
