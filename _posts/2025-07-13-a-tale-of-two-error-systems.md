---
tags: [exception, error, SimpleXml]
categories: [php]
---

## A tale of two error systems

Legacy php offers a lot of surprises, one of them is there there are two different error systems the legacy: [`trigger_error()`](https://www.php.net/manual/en/function.trigger-error.php), which has these categories of error:  `E_USER_ERROR, E_USER_WARNING, E_USER_NOTICE, E_USER_DEPRECATED` and is handled by registering handlers with [`set_error_handler()`](https://www.php.net/manual/en/function.set-error-handler.php). And the modern php try catch Exception/Error system. Yes don’t confuse an "error" with a `Error` (which is a `Throwable`).
Each is entirely transparent to the other (which is probably for the best) and you should really only be using one or the other to avoid confusion. Something that doesn’t do this is the core php library `SimpleXml`, pass it an invalid xml and it throws both a modern `Exception` and an old style error. This lead to some head scratching in some code I thought was safe in a try catch guard:

```php
    try {
        new SimpleXml($request);
    // some logic to redact api key
    } catch (Exception $e) {
        $this->logging->warning("couldn’t add redacted response to the error context");
    }
```

This code was still blowing up in a bad way, however it was coming through the old handler. The fix turned out to be simple, the oft misused error suppression operator `@`:

```php
    try {
        @new SimpleXml($request);
    // some logic to redact api key
    } catch (Exception $e) {
        $this->loggin->warning("couldn’t add redacted response to the error context");
    }
```

this turns out to suppress old errors and not the new Exception/Error. Still definitely one to use with caution and with unit tests!