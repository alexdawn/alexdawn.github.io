---
tags: [php-fpm, ajax, session]
categories: [php]
---

## Freeing php from the session lock

So on a project I took a slow page broke up the elements and queries into a bunch of ajax "fragments"
this is pretty simple to do with jquery. However this did not result in a speed up for a given user. This was because
the default way php is called is directly from apache, the fix for that is using `php-fpm`.

`php-fpm` uses something called fast common gateway interface fCGI so that apache is able request php to run code from
a waiting session that already is running and importantly has hopefully the right files autoloaded and the right bytecode right in memory. This also frees up apache to start severing the next ajax request.

That was the theory but in practice having made these changes I was still seeing concurrent requests.

The next thing I came across was the php session and how the default session stores this data in a file and importantly
has a file lock which was probably what was stopping the ajax requests from processing all at once.

However php comes with a function you can call to register alternative callbacks for working with a session and with that it is possible to supply an alternative option such as `memcache`.