## How to test scripts in phpunit (alternative way)

This is presented after the previous way as it requires a bit more of a heavy refactor that the way shown in part 4. This was is wrapping the script into a class and adding a python style `if __name__ === ‘__main__’:` at the bottom. This is a very common convention in python.

This is a very common convention in python in a way to cleanly and safely make a script both have declarations as to be autoloaded and used in other code (e.g. a phpunit test), while still being invokable as a script.

PSR1 quite wisely says that all code files should either declare code or run code never both. This is to avoid the evils of unexpected side-effects, it would be some seriously nasty code that purely the act of loading the class also executed some other code. Sadly this is all too common in legacy scripts. This is what the conditional execution solves, if the script is invoked as the initial script that it’s quite clear that code should be run and not just loaded, while it is safe from being run if some other script loads it in not as the first script, for example by phpunit looking to run a single method.

While this doesn't quite follow PSR1 strictly interpreted, as when run directly as a script it at least respects the spirit of the rule.

```php
// MyOldScript.php
<?php

declare(strict_types = 1);

class MyOldScript
{
    public function run()
    {
        ...
        $this->someFunctionIWantToTest();
        ..
    }

    public function someFunctionIWantToTest()
    {
        ...
    }
}

// the below runs if this file is ran php ./MyOldScript.php
// but not if it was loaded via include/require
if (isset($argv) && $argv[0] === basename(__FILE__)) {
    (new MyOldScript())->run();
}
```