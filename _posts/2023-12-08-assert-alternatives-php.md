---
layout: post
title:  "Alternatives to assert() function in PHP"
---

The `assert` function is a very convenient way to validate data in PHP, but is
this function reliable ? The answer is no. The thing with `assert` function is
that it can be disabled, so it's usually disabled in production environments.

## How does it work?

`Assert` function allows you to evaluate an _expression_, if the
condition is _true_ then the execution will continue normally, but if the
condition is _false_ then an `AssertionError` will be thrown.

```php
$name = false;
assert(is_string($name)); // will throw AssertError
```

If you want to make this example work you must have the following `php.ini`
configuration:

```ini
zend.assertions = 1
assert.active = 1
```

For more details please refer
to [assert documentation](https://www.php.net/manual/en/function.assert.php).

## Assert alternatives

Since I cannot use assert function in production, I had to found a replacement.
I present you three alternatives I found.

**Alternative 1: use if statement**

You can mimic assert behaviour using a simple if statement, and negating the
condition.

```php
$name = false;
if (!is_string($name)){
    throw new \Exception('Name must be string')
}
```

On one hand this solution is very intuitive, on the other hand it's also
verbose, now we need three lines of code instead of only one.

**Alternative 2: write your own function**

To be closer to the original `assert` function I wrote my own function, of
course it can be used in any environment.

```php
use function Jawira\TheLostFunctions\throw_unless;

$name = false;
throw_unless(is_string($name), new \Exception('Name must be string'));
```

The downside of this solution if that you have to install an external library,
in this
case [jawira/the-lost-functions](https://packagist.org/packages/jawira/the-lost-functions)
.

**Alternative 3: use ternary operator**

Since PHP 8.1, throw is an expression, I took advantage of this feature to write
a one-liner `assert` equivalent:

```php
$name = false;
is_string($name) ?: throw new \Exception('Name must be string');
```

This is the best alternative. It takes a single line of code, and it only uses
vanilla PHP - no external library is involved - therefore this is the solution I
use every day in my projects.

## Conclusion

Assert function is only meant to be used in dev environment, and it will be very
likely disabled in production. As a replacement you can use any
of the alternatives I proposed, being the last one my preferred.
