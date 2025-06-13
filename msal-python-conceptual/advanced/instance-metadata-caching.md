---
title: Instance metadata caching
description: "Every developer wants their program to run faster. In this article, we are going to show you how you can add a one-liner to make your MSAL Python powered app to acquire a token roughly 1.5x to 2x faster!"
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

# Instance metadata caching

Every developer wants their program to run faster. In this article, we are going to show you how you can add a one-liner to make your MSAL Python powered app to acquire a token roughly 1.5x to 2x faster!

## Preparation

1. First let's pick an existing sample script to demonstrate this experiment.
We choose the [username password sample](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.0.0/sample/username_password_sample.py),
because it would require no user interaction,
so that our time measurement would not be affected by slow human interaction.

2. Configure a config.json for your sample per
   [instruction](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.0.0/sample/username_password_sample.py#L2-L15).

3. In order to time it, you would want to add this optional snippet at the very beginning of your sample:

   ```python
   import time; t = time.time()
   ```

   and then you add this line at the very end of your sample:

   ```python
   print("It took", time.time() - t, "second(s)")
   ```

   Ideally, you would want to comment out
   [these lines of actual Microsoft Graph API call](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.0.0/sample/username_password_sample.py#L62-L65),
   because it is not part of token acquisition. 
   Lastly, you can optionally also uncomment
   [these two lines](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.0.0/sample/username_password_sample.py#L31-L32)
   to enable logging for you to see what happens under the hood.

## Before

Now run it several times. You will like see an output like this:

   ```bash
   $ python username_password_sample.py config.json
   It took 0.9160797595977783 second(s)

   $ python username_password_sample.py config.json
   It took 0.7800290584564209 second(s)

   $ python username_password_sample.py config.json
   It took 0.8385567665100098 second(s)
   ```

   So it took roughly 0.8+ seconds to acquire a token.

## Actual Magic

Now let's have some fun.

Install a 3rd-party module "requests-cache" by running this once: `pip install requests-cache`,
and then add this line at the very beginning of your sample:
```python
import requests_cache; requests_cache.install_cache()
```

## After

Now run it several times. You will like see an output like this:

   ```bash
   $ python username_password_sample.py config.json
   It took 0.41900205612182617 second(s)

   $ python username_password_sample.py config.json
   It took 0.480008602142334 second(s)

   $ python username_password_sample.py config.json
   It took 0.44204020500183105 second(s)
   ```

So it took roughly 0.4+ seconds to acquire a token.

## What is going on?

Before acquiring a token, MSAL would need to hit several discovery endpoints to find some metadata.
These metadata are infrequently changed, so they can be cached and reused.
All these are standard practice happening in the HTTP layer,
that's why MSAL Python does not have to implement such behavior by itself,
and a generic HTTP cache library would do this job for us.

In our case, MSAL Python already uses a popular HTTP library named `requests`,
and then there is another generic library `requests-cache`
designed to monkey-patch the `requests` and provide HTTP cache behaviors, automatically.
Even better, its default behavior is to persist it on disk,
so our command-line sample still benefits from such HTTP cache,
dispite running as a new process every time.

In fact, this trick does not just help MSAL Python,
it improves all other cache-able HTTP traffic of your app.

## Conclusion

Now you have it, a whopping 1.5x to 2x performance gain by only one single line of code.
And it is universally applicable to all your apps utilizing HTTP.

Happy Hacking!
