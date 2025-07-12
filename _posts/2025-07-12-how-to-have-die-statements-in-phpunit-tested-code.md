## How to have die statements in phpunit tested code

This is the first of a 5 part guide of how to make light touch changes to improve your experience with working with legacy php.

Old scripts might make liberal use of the exit/die directive in php. This is a problem if you are trying to test your code with phpunit as it will kill the test suite itself (unless you are using the `runInSubprocess` directive which might not be an option for you, even then it’s not ideal as the message php unit gives when the sub process dies is the unhelpful "the test ended unexpectedly"). The normal option would be to upgrade to throwing some kind of exception.

> **_Aside:_** avoid throwing the base `Exception` class itself, always use a subclass, otherwise your handling code will have to handle `Exception`, but is your handler really setup to handle any and all exceptions to come it’s way? I didn’t think so. Make sure to create and use a specific Exception child class that describes your situation.

However I was in the unfortunate position that such an option was seemingly not available, an unhandled exception in prod would result in emailing the whole monitoring team. My initial option was then to have a method that would except or die depending on the environmental context, an okay option but kind felt clunky to have this utility method.

The next brain wave, was can you exit a php process inside a constructor? Yes you can, here is is how you can continue making use of exit without killing your phpunit test suite:

```php
class ExitException extends Exception
{
    public function __construct(string $message = '', int $code = 0, Throwable $previous) {
        // or however you determine what environment you are running
         if ($_ENV['env'] !== 'test') {
              exit($code ?: $message); // use the exception code as the exit code, also if code isn’t zero, make sure it is passed not any message as the php docs for exit says a string message is treated as exit zero.
         }
         parent::__construct($message, $code, $previous);
    }
}
```
