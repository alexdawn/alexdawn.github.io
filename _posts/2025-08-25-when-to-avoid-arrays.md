---
tags: [php, best-practice, typing, data-structures]
categories: [php]
---

## When to avoid arrays

At a place I used to work they had a rule for python; don't use arrays in production code to pass around data between functions and methods. Make a `NamedTuple` instead. The reasoning is simple an array you can squeeze anything you like in and while at first it may seem sensible, made it one place and consumed in another, over time you'll find these packages of data added onto in more than one place. No longer is it possible to see in one place all the potential keys that may be present. Will the key always be there? Then your IDE is also silent on the matter, unless you are lucky enough to have the phpstan [array shapes](https://phpstan.org/writing-php-code/phpdoc-types#array-shapes) telling you what to expect but even then it is only good if the comments and documentation is kept up to date.

```php
 return [
    'title' => 'Dune',
    'author' => 'Frank Herbert',
    'publicationYear' => 1965,
    'isbn' => 9780425027066
 ];
```

Quite aside from the above issues, nothing in the snippet actually says the array represent a book, often you'll see arrays and you have to infer what is it from context, often easy enough with literals but often an array created from other variables.

If you are not familiar with python or NamedTuples, then in short they are a data-structure than has a fixed number of properties with a given type, instead of a basic tuple which you can think of as a fixed length array a named tuple you can access not just by positional index but by name of the property. Since the properties are declared an IDE is able to help autocomplete and alert if a property is missing, misspelt or is being given the wrong data type. It is also immutable because you cannot have too much of a good thing.

You'd want to use them in the same places as you'd use "Data Transfer Objects" and "Value Objects".

An oft used argument is arrays are used because they are quick and easy to make, which is why the rule above makes the exception for prototypes, however the extra effort for something better is not excessive and will pay dividends in better structured production code.

Something vaguely like a named tuple in php would be something like:

```php
final class Book
{
     public function __construct(
        public readonly string $title,
        public readonly string $author,
        public readonly int $publicationYear,
        public readonly int $isbn
     ) {
     }
}
```

Pretty snappy eh? We get the immutability from the `readonly` modifier. Then via constructor promotion and the public keyword we avoid all of the boilerplate of a normal class and can skip the usual getter/setters. Normally properties are made private for good reason, however with the read-only that gives the extra protection that there will be no badly behaved code that messes with the properties. All of the above means you can easily access data from your object e.g. `$book->title`, the object can't be misused in future to pack in additional properties that do not belong there and your IDE can tell you if you are trying to access a property that does not exist.