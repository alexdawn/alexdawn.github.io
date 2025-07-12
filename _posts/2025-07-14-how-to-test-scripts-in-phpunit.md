## How to test scripts in phpunit

Phpunit is well suited to tested class based code, but how do we get it to test legacy code? Especially if that script does all sorts of strange and nasty things, [breaking PSR1 by mixing declaration and actions](https://www.php-fig.org/psr/psr-1/), like printing to stdout, just dying on you (see part 1), and using the dreaded globals.

```php
// stick this method inside your phpunit test class
public function runScript(...$args): string
{
     global $usedInScript;  // declare your globals as the script is no longer being run in global scope
     $somethingElseUsedInScript = $args[0]; // or override them for the test
     ob_start();
     try {
         require($path);
     } finally {
         $response = ob_get_clean();
     }
     return $response;
}
```

okay so what is going on here? The `ob_*` functions allow us to capture anything headed to standard out from an echo and instead direct it to the variable (which we can then do asserts on).

Why the try finally? Phpunit marks your test as risky if it doesn’t leave the output buffer the level the same as when it starts and ends. When does that happen? If your test script throws an exception (deliberately as part of your test for exceptional behaviour) it will never get to `ob_get_clean()` unless it’s in an exception or finally block. I’ve picked finally because that’s slightly cleaner than a:

```php
catch (Exception) {
    ob_get_clean();
}
return ob_get_clean();
```

the other big gotcha is the location of the `require` changes the scope of the required code, your script might rely on globals (registered via a `bootstrap.php` script, which you can also instruct phpunit to bootstrap with), so the function with have to declare them as in scope, or just declare local mock versions with the same name.