---
title: Logging
description: "Logging in MSAL Python is designed to use the standard Python logging mechanisms, so all your previous knowledge of Python logging applies to MSAL Python."
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

# Logging

Logging in MSAL Python is designed to use the standard Python logging mechanisms, so all your previous knowledge of Python logging applies to MSAL Python.

* By default, the logging in any Python script is turned off. If you want to enable debug logging for ALL modules in your entire Python script, you use `logging.basicConfig(level=logging.DEBUG)`.
* Most of the MSAL Python logs are already in debug level, which would be turned off by default. But if you want to enable debug logging to debug the OTHER modules in your Python script, therefore want to silence MSAL, you simply turn off the logger used by MSAL Python: `logging.getLogger("msal").setLevel(logging.WARN)`.
* MSAL Python does not log Personal Identifiable Information (PII). So there is not even a turn-on-PII-logging toggle in MSAL Python. App developers could still use standard Python logging to log whatever content. By doing so, the app takes responsibility for safely handling highly sensitive data and following regulatory requirements.

Any further reading on Python logging topic, please refer to [Python standard logging tutorial](https://docs.python.org/3/howto/logging.html#logging-basic-tutorial).
