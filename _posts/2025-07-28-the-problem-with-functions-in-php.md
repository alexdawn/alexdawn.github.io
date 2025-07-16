## The problem with functions in php

I like functions because I like functional programming. 

You can namespace and use functions with:

```php
use function Org\Project\Path\To\My\FunctionFolder;
```

The problem with  function in php is they are left behind compared with objects when it comes to lazy-loading and are not supported by autoloaders, the only type of autoloader composer works for functions is the file autoloader which isn’t really much of an autoloader at all as it eager loads all files at startup.

Until we can load functions with autoloader’s I will just have to load function as static methods:
```php
namespace Org\Project\Path\To\My\FunctionFolder;

abstract class FileName
{
    public static function function1() {
       …
    }
}

// usage:
use Org\Project\Path\To\My\FunctionFolder\FileName;

FileName::function1();

```

Not ideal as static functions can be heavily misused, difficult to mock in tests and are a right nightmare for anything stateful.