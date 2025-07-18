---
title: Using MSAL Python with an Auth Broker on Linux
description: MSAL is able to call Microsoft Single Sign-on for Linux, which is a component that ships as a dependency of Intune Portal. This component acts as an authentication broker allowing the users of your app to benefit from integration with accounts known to the broker.
author: ploegert
ms.author: jploegert
ms.service: msal
ms.topic: how-to
ms.date: 06/03/2025
---

# Enable SSO in native Linux using MSAL Python

Microsoft Authentication Library (MSAL) is a Software Development Kit (SDK) that enables apps to call the Microsoft Single Sign-on to Linux broker, a Linux component that is shipped independent of the Linux Distribution, however it gets installed using a package manager using `sudo apt install microsoft-identity-broker` or `sudo dnf install microsoft-identity-broker`.

This component acts as an authentication broker, allowing the users of your app to benefit from integration with accounts known to Linux - such as the account you signed into your Linux sessions for apps that consume from the broker.

The broker is also bundled as a dependency of applications developed by Microsoft (such as [Company Portal](/mem/intune-service/user-help/enroll-device-linux))). An example of installation of the broker being installed is when a Linux computer is enrolled into a company's device fleet via an endpoint management solution like [Microsoft Intune](/mem/intune/fundamentals/what-is-intune).

## What is a broker

An authentication broker is an application that runs on a user’s machine that manages the authentication handshakes and token maintenance for connected accounts. The Linux operating system uses the Microsoft single sign-on for Linux as its authentication broker. It has many benefits for developers and customers alike, including:

- **Enables Single Sign-On**: enables apps to simplify how users authenticate with Microsoft Entra ID and protects Microsoft Entra ID refresh tokens from exfiltration and misuse
- **Enhanced security.** Many security enhancements are delivered with the broker, without needing to update the application logic.
- **Feature support.** With the help of the broker developers can access rich OS and service capabilities.
- **System integration.** Applications that use the broker plug-and-play with the built-in account picker, allowing the user to quickly pick an existing account instead of reentering the same credentials over and over.
- **Token Protection.** Microsoft single sign-on for Linux ensures that the refresh tokens are device bound.

## How to opt in to use broker?

1. In the MSAL Python library, we've introduced the `enable_broker_on_linux` flag, which enables the broker on both WSL and standalone Linux.
    - If your goal is to enable broker support solely on WSL for Azure CLI, you can consider modifying the Azure CLI app code to activate the `enable_broker_on_wsl` flag exclusively on WSL.
    - If you are writing a cross-platform application, you will also need to use `enable_broker_on_windows`, as outlined in the [Using MSAL Python with Web Account Manager](wam.md) article.
    - You can set any combination of the following opt-in parameters to true:

| Opt-in flag              | If app will run on                | App has registered this as a Desktop platform redirect URI in Azure Portal       |
| ------------------------ | --------------------------------- | -------------------------------------------------------------------------------- |
| enable_broker_on_windows | Windows 10+                       | ms-appx-web://Microsoft.AAD.BrokerPlugin/your_client_id                          |
| enable_broker_on_wsl     | WSL                               | ms-appx-web://Microsoft.AAD.BrokerPlugin/your_client_id                          |
| enable_broker_on_mac     | Mac with Company Portal installed | msauth.com.msauth.unsignedapp://auth                                             |
| enable_broker_on_linux   | Linux with Intune installed       | `https://login.microsoftonline.com/common/oauth2/nativeclient` (MUST be enabled) |

2. Your application needs to support broker-specific redirect URIs. For `Linux` specifically, the URL for the redirect URI must be:

    ```text
    https://login.microsoftonline.com/common/oauth2/nativeclient
    ```

1. To use the broker, you will need to install the broker-related packages in addition to the core MSAL from PyPI:

    ```python
    pip install "msal[broker]>=1.33.0b1,<2"
    ```
    
4. Once configured, you can call `acquire_token_interactive` to acquire a token.

    ```python
    result = app.acquire_token_interactive(["User.ReadBasic.All"],
                        parent_window_handle=app.CONSOLE_WINDOW_HANDLE)
    ```

## Parameters for broker support

The following parameters are available to configure broker support in MSAL Python. These parameters can be passed to the `PublicClientApplication` constructor or to the `acquire_token_interactive` method.

| Parameters: | Type | Description |
|-------------|------|-------|
| enable_broker_on_windows | `boolean` | This setting is only effective if your app is running on Windows 10+. This parameter defaults to None, which means MSAL will not utilize a broker. </br></br>`New in MSAL Python 1.25.0.` |
| enable_broker_on_wsl | `boolean` | This setting is only effective if your app is running on WSL. This parameter defaults to None, which means MSAL will not utilize a broker. </br></br>`New in MSAL Python 1.25.0`. |
| enable_broker_on_mac | `boolean` | This setting is only effective if your app is running on Mac with Company Portal installed. This parameter defaults to None, which means MSAL will not utilize a broker. </br></br>`New in MSAL Python 1.31.0`.|
| enable_broker_on_linux | `boolean` | This setting is only effective if your app is running on Linux with Intune installed. This parameter defaults to None, which means MSAL will not utilize a broker. </br></br>`New in MSAL Python 1.33.0`. |
| parent_window_handle | `int` | <i>OPTIONAL</i></br></br>

### Notes regarding parent_window_handle

The `parent_window_handle` parameter is required even though on Linux it is not used. For GUI applications, the login prompt location will be determined ad-hoc and currently cannot be bound to a specific window. In a future update, this parameter will be used to determine the _actual_ parent window.

| Condition | Description |
|---|---|
|App does not want to utilize a broker|no need to specify a parent_window_handle|
|App opts to use a broker|parent_window_handle is required|
|App is a GUI app running on Windows or Mac system|required to provide its window handle, so that the sign-in window will pop up on top of your window|
|App is a console app running on Windows or Mac system|can use a placeholder `PublicClientApplication.CONSOLE_WINDOW_HANDLE`|
|App is intended to be a cross-platform application| App needs to use `enable_broker_on_windows`, as outlined in the [Using MSAL Python with Web Account Manager](wam.md) article.|

## The fallback behaviors of MSAL Python’s broker support

MSAL will either error out, or silently fallback to non-broker flows.

1. MSAL will ignore the enable_broker_… and bypass broker on those auth flows that are known to be NOT supported by broker. This includes ADFS, B2C, etc.. For other “could-use-broker” scenarios, please see below.

2. MSAL errors out when app developer opted-in to use broker but a direct dependency “mid-tier” package is not installed. Error message guides app developer to declare the correct dependency msal[broker]. We error out here because the error is actionable to app developers.

3. MSAL silently “deactivates” the broker and fallback to non-broker, when opted-in, dependency installed yet failed to initialize. We anticipate this would happen on a device whose OS is too old or the underlying broker component is somehow unavailable. There is not much an app developer or the end user can do here. Eventually, the conditional access policy shall force the user to switch to a different device.

4. MSAL errors out when broker is opted in, installed, initialized, but subsequent token request(s) failed.

>[!IMPORTANT]
>If broker-related packages are not installed and you will try to use the authentication broker, you will get an error: `ImportError: You need to install dependency by: pip install "msal[broker]>=1.xx,<2"`.

<!--
This one is an error message whose content happens to be changing based on platform. The doc shall just use a placeholder that is good enough.
-->
>[!NOTE]
>The `parent_window_handle` parameter is required even though on Linux it is not used. For GUI applications, the login prompt location will be determined ad-hoc and currently cannot be bound to a specific window. In a future update, this parameter will be used to determine the _actual_ parent window.

## Token caching

The authentication broker handles refresh and access token caching. You do not need to set up custom caching.

## Building an example app

You can find a sample app that demonstrates how to use MSAL Python with the authentication broker on Linux in the [MSAL Python GitHub repository](https://github.com/AzureAD/microsoft-authentication-library-for-python/). The sample app is located in the `samples/console_app` directory and includes examples of how to use the broker for authentication.

### **App Registration**

Update your App registration in the Azure portal to include the broker-specific redirect URI for Linux:

```text
https://login.microsoftonline.com/common/oauth2/nativeclient
```

### **Linux Dependencies**

First, check if you have python3 installed on your Linux distribution.

```bash
python3 --version
```

If not, install it using the package manager for your distribution.

#### [Ubuntu](#tab/ubuntudep)

To install on debian/Ubuntu based Linux distribution:

```bash
sudo apt install python3 python3-pip -y
```

#### [Red Hat Enterprise Linux](#tab/rheldep)

To install on Red Hat/Fedora based Linux distribution:

```bash
sudo dnf install python3 python3-pip -y
```

---

### **Python Dependencies**

To use the broker, you will need to install the broker-related packages in addition to the core MSAL from PyPI:

```python
pip install "msal[broker]>=1.33.0b1,<2"
```

### Create Project
Once configured, you can call `acquire_token_interactive` to acquire a token.

```python
import sys  # For simplicity, we'll read config file from 1st CLI param sys.argv[1]
import json
import logging
import requests
import msal

# Optional logging
# logging.basicConfig(level=logging.DEBUG)

var_authority = "https://login.microsoftonline.com/common"
var_client_id = "your-client-id-here"  # Replace with your app's client ID
var_username = "your-username-here"  # Replace with your username, e.g., "
var_scope = ["User.ReadBasic.All"]
# Removed unused variable to avoid confusion


# Create a preferably long-lived app instance which maintains a token cache (Default cache is in memory only).
app = msal.PublicClientApplication(
    var_client_id, 
    authority=var_authority,
    enable_broker_on_windows=True,
    enable_broker_on_wsl=True
    )

# The pattern to acquire a token looks like this.
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=var_username)
if accounts:
    logging.info("Account(s) exists in cache, probably with token too. Let's try.")
    result = app.acquire_token_silent(var_scope, account=accounts[0])

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    
    result = app.acquire_token_interactive(var_scope,parent_window_handle=app.CONSOLE_WINDOW_HANDLE)
    
if "access_token" in result:
    print("Access token is: %s" % result['access_token'])

else:
    print(result.get("error"))
    print(result.get("error_description"))
    print(result.get("correlation_id"))  # You may need this when reporting a bug
    if 65001 in result.get("error_codes", []):  # Not mean to be coded programatically, but...
        # AAD requires user consent for U/P flow
        print("Visit this to consent:", app.get_authorization_request_url(config["scope"]))
```
