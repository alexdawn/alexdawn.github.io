---
tags: [autoloader, alias, refactoring, joomla]
categories: [php]
---

## How to start namespacing legacy code with Namespace alias

Now this was a tricky problem and initial googling didn’t show up anything, how do you register a class with multiple namespaces, the exact same class? First of all why would you want to?

Okay so picture this you’ve an enormous legacy project with everything being loaded in with a custom autoloader into the global namespace. You’ve added a composer PSR4 autoloader which works for new classes, but moving over existing classes is a nightmare, when you make the switch you’ve then got to find every file that uses it and add in use statements, an absolute spiders web of a problem, the ideal solution would be our `LegacyClass` both be in the global space but also in the new namespace `Org/Project/Path/LegacyClass`.

Stumbled across that’s exactly what they do with Joomla, the old code everything was prefixed with J, `Jkernel` etc... to try and reduce the chance of a namespace clash, they’ve moved over to namespaced names but the old ones still work thanks to aliasing, and I’d finally found the function google had failed to show: `class_alias()`.

Now our problem isn’t yet over, as it comes `class_alias()` is a bit too eager and that function causes the autoloader to fire, not ideal if your alias map causes your entire project to eager load. The fix for this is call the `class_alias` inside your own custom autoloader. Now this has yet more problems, because there are two ways your class will get loaded:
* via the original name now aliased
* via the new PSR4 fully namespaced names
and the alias registration has to work either way around.

To ensure your customer autoloader has a chance to register an alias (it won’t be called if the PSR4 autoloader returns true) it has to be prepended to the autoload list, then return false for no match so php will continue onwards towards the PSR4 autoloader. However it also cannot let the legacy custom autoloader fire and load the class incorrectly without registering the alias, so this method cannot just return false and let php itself work the autoload list, nope it has to managed this itself as after autoloading the class only then can the alias call be made

```php
class AliasAutoloader
{
    private static array $aliasMap = [
        MyLegacyClass::class => Org\Project\Path\MyLegacyClass::class,
        ...
    ];
    /**
     * @return bool autoload callbacks return true if found false if the next loader should run
     */
    public static load(string $className): bool
    {
        // check if class is the alias or is aliased
        if (!in_array($className, static::aliasMap)) {
            return false; // normal autoloading
        }
        $fullyQualifiedClassname = static::aliasMap[$className];
        // if aliased need find the normal way to load it
        // call the next functions ourselves instead of returning false as we have additional work to do
        foreach (spl_autoload_functions() as $function) {
            // don't recursively call itself!
            if ($function !== [static, 'load'] && $function($fullyQualifiedClassname)) {
                break;
            }
            // problem couldn't find the right autoloader for this class!
            throw new RuntimeException("problem couldn't find the right autoloader for this class!");
        }
        // then register the alias, this has to be done after the class is loaded in the foreach above or this will recursively trigger an autoload
        class_alias($fullyQualifiedClassname, $className);
        return true; // already called the functions and autoloaded no need to call any more
    }
}

// make sure this is is called first or register_alias may fail
spl_autoload_register(AliasAutoloader::load(...), prepend: true);
```