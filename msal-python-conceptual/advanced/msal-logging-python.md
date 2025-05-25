---
title: Logging errors and exceptions in MSAL for Python
description: Learn how to log errors and exceptions in MSAL for Python
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

# Logging in MSAL for Python

The Microsoft Authentication Library (MSAL) apps generate log messages that can help diagnose issues. An app can configure logging with a few lines of code, and have custom control over the level of detail and whether or not personal and organizational data is logged. We recommend you create an MSAL logging implementation and provide a way for users to submit logs when they have authentication issues.

Logging in MSAL Python is designed to use the standard Python logging mechanisms, so all your previous knowledge of Python logging applies to MSAL Python.

* By default, the logging in any Python script is turned off. If you want to enable debug logging for ALL modules in your entire Python script, you use `logging.basicConfig(level=logging.DEBUG)`.
* Most of the MSAL Python logs are already in debug level, which would be turned off by default. But if you want to enable debug logging to debug the OTHER modules in your Python script, therefore want to silence MSAL, you simply turn off the logger used by MSAL Python: `logging.getLogger("msal").setLevel(logging.WARN)`.
* MSAL Python does not log Personal Identifiable Information (PII). So there is not even a turn-on-PII-logging toggle in MSAL Python. App developers could still use standard Python logging to log whatever content. By doing so, the app takes responsibility for safely handling highly sensitive data and following regulatory requirements.


## Logging levels

MSAL provides several levels of logging detail:

- `LogAlways`: No level filtering is done on this log level. Log messages of all levels will be logged.
- `Critical`: Logs that describe an unrecoverable application or system crash, or a catastrophic failure that requires immediate attention.
- `Error`: Indicates something has gone wrong and an error was generated. Used for debugging and identifying problems.
- `Warning`: There hasn't necessarily been an error or failure, but are intended for diagnostics and pinpointing problems.
- `Informational`: MSAL will log events intended for informational purposes not necessarily intended for debugging.
- `Verbose` (Default): MSAL logs the full details of library behavior.

> [!NOTE]
> Not all log levels are available for all MSAL libraries.

## Personal and organizational data

By default, the MSAL logger doesn't capture any highly sensitive personal or organizational data. The library provides the option to enable logging personal and organizational data if you decide to do so.

The following sections provide more details about MSAL error logging for your application.

## MSAL for Python logging

Logging in MSAL for Python leverages the [logging module in the Python standard library](https://docs.python.org/3/library/logging.html). You can configure MSAL logging as follows (and see it in action in the [username_password_sample](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.0.0/sample/username_password_sample.py#L31L32)):

### Enable debug logging for all modules

By default, the logging in any Python script is turned off. If you want to enable verbose logging for **all** Python modules in your script, use `logging.basicConfig` with a level of `logging.DEBUG`:

```python
import logging

logging.basicConfig(level=logging.DEBUG)
```

This will print all log messages given to the logging module to the standard output.

### Configure MSAL logging level

You can configure the logging level of the MSAL for Python log provider by using the `logging.getLogger()` method with the logger name `"msal"`:

```python
import logging

logging.getLogger("msal").setLevel(logging.WARN)
```

### Configure MSAL logging with Azure App Insights

Python logs are given to a log handler, which by default is the `StreamHandler`. To send MSAL logs to an Application Insights with an Instrumentation Key, use the `AzureLogHandler` provided by the `opencensus-ext-azure` library.

To install, `opencensus-ext-azure` add the `opencensus-ext-azure` package from PyPI to your dependencies or pip install:

```console
pip install opencensus-ext-azure
```

Then change the default handler of the `"msal"` log provider to an instance of `AzureLogHandler` with an instrumentation key set in the `APP_INSIGHTS_KEY` environment variable:

```python
import logging
import os

from opencensus.ext.azure.log_exporter import AzureLogHandler

APP_INSIGHTS_KEY = os.getenv('APP_INSIGHTS_KEY')

logging.getLogger("msal").addHandler(AzureLogHandler(connection_string='InstrumentationKey={0}'.format(APP_INSIGHTS_KEY)))
```

### Personal and organizational data in Python

MSAL for Python does not log personal data or organizational data. There is no property to turn personal or organization data logging on or off.

You can use standard Python logging to log whatever you want, but you are responsible for safely handling sensitive data and following regulatory requirements.

For more information about logging in Python, please refer to Python's  [Logging: how-to](https://docs.python.org/3/howto/logging.html#logging-basic-tutorial).