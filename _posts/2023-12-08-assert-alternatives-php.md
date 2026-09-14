---
layout: post
title: "Alternatives to the assert() function in PHP"
---

The `assert` function is a very convenient way to validate data in PHP. But is
this function reliable? The answer is no. The issue with the `assert` function
is that it can be disabled. It is meant only for development environments and is
usually disabled in production environments.

## How does it work?

The `assert` function allows you to evaluate an _expression_. If the condition
is _true_, execution continues normally. But if the condition is _false_, an
`AssertionError` is thrown.

```php
$name = false;
assert(is_string($name)); // will throw AssertionError
```

To make this example work, you need the following `php.ini` configuration:

```ini
zend.assertions = 1
assert.active = 1
```

For more details, see
the [assert documentation](https://www.php.net/manual/en/function.assert.php).

## Assert alternatives

Since I cannot use the `assert` function in production, I had to find a
replacement. Here are three alternatives I found:

**Alternative 1: Use an If Statement**

You can mimic the `assert` behavior using a simple `if` statement and negating
the condition.

```php
$name = false;
if (!is_string($name)) {
    throw new \Exception('Name must be string');
}
```

On the one hand, this solution is very intuitive. On the other hand, it is also
verbose: we now need three lines of code instead of only one.

**Alternative 2: Write Your Own Function**

To be closer to the original `assert` function, I wrote my own function; of
course, it can be used in any environment.

```php
use function Jawira\TheLostFunctions\throw_unless;

$name = false;
throw_unless(is_string($name), new \Exception('Name must be string'));
```

The downside of this solution is that you have to install an external library:
in this
case, [jawira/the-lost-functions](https://packagist.org/packages/jawira/the-lost-functions).

**Alternative 3: Use the Ternary Operator**

Since PHP 8.1, the `throw` keyword is an expression. I used this feature to
write a one-liner `assert` equivalent:

```php
$name = false;
is_string($name) ?: throw new \Exception('Name must be string');
```

This is the best alternative. It uses only a single line of code with vanilla
PHP, so no external library is involved. This is the solution I use daily in my
projects.

**Update (2026)**

However, using single-line ternary operators, as suggested in this post, is not
compatible with Psalm.

The following two lines of code will not be analyzed correctly by Psalm:

```php
// Using ternary operator
is_numeric($number) ?: throw new Exception('Must provide a number.');
// Using null coalescing operator
$entity ?? throw new Exception('Variable must be defined.');
```

These lines will generate the following error:

```console
INFO: MixedOperand - 14:10 - Left operand cannot be mixed
```

As a solution, the ternary operator can be replaced with the `or` operator. This
makes your code compatible with Psalm and improves readability:

```php
is_numeric($number) or throw new Exception('Must provide a number.');
```

Source: <https://github.com/vimeo/psalm/issues/10673>

## Conclusion

The `assert` function is only meant to be used in a development environment and
will likely be disabled in production. As a replacement, you can use any of the
alternatives I proposed, with the last one being my favorite.
