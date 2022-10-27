---
layout: post
title: "My favorite PHP feature (and it's broken)"
---
## The problem

Increment operator should not be used, the reason is that there can be collisions with other ways to represent numbers.

Let's see an example:

@todo show example

What happened ? The reason is that PHP you can also specify integers like this:

- "5e3" // 5000.0
- You can validate this with is_numeric("5e3"); // true

Usually, you would write "5e3" without quotes, but it will also work with quotes!

In PHP8 you can also have

## Conclusion

This functionality should be seen as a curiosity, but due to its flaws it should not be used.

## Resources

- [Incrementing/Decrementing Operators](https://www.php.net/manual/en/language.operators.increment.php)
