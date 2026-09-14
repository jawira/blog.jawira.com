---
layout: post
title: "Profiling a PHP script with Xdebug"
---

Software profiling is a powerful technique for optimizing your code. Among other
things, a profile snapshot will show you which parts of your software are the
slowest.

In this article, I will explain how to create a profile snapshot for a PHP
script.

## Requirements

I assume you are using _Ubuntu_ and have PHP already installed. Also install
_Xdebug 3_ and _KCachegrind_:

```console
$ apt install php-xdebug
$ apt install kcachegrind
```

## Creating a profile file

As an example, let's profile tests
from [jawira/plantuml-encoding](https://github.com/jawira/plantuml-encoding).

```console
$ git clone https://github.com/jawira/plantuml-encoding.git
$ cd plantuml-encoding
$ php tests/vanilla.php
```

To profile `tests/vanilla.php`, execute the following command:

```console
$ php -dxdebug.mode=profile -dxdebug.output_dir=. tests/vanilla.php
```

After execution, a profile snapshot is created with the naming pattern
`cachegrind.out.xxxxx` (where `xxxxx` is a number).

![Terminal screenshot showing profile file creation](/images/profiling_xdebug_terminal.png)

_Xdebug_ can be configured in `php.ini`. Here, we configured it on-the-fly by
passing `php.ini` settings through the terminal.

* **`-dxdebug.mode=profile`**: This option enables profile mode for this
  command.
* **`-dxdebug.output_dir=.`**: Saves profile snapshots in the current directory
  (`.`); otherwise, they are saved in `/tmp`.

In our case, profiling mode is ephemeral. However, this can become problematic
if you activate profiling by other means, such as directly editing the `php.ini`
file. Profile snapshots can consume a lot of disk space, so never leave profile
mode permanently enabled.

## Opening the profile file

To visualize the profile snapshot, simply open it (in our case
`cachegrind.out.14476`) with _KCachegrind_.

![KCachegrind screenshot showing profile analysis](/images/profiling_xdebug_kcachegrind.png)

Reading and interpreting profiling files is beyond the scope of this article.
However, I recommend
watching [this video](https://www.youtube.com/watch?v=h-0HpCblt3A) presenting
KCachegrind.

## Conclusion

Creating a profile snapshot is easy once you know how to configure _Xdebug_
properly. Using `-d` options is a very convenient technique for enabling
profiling from the terminal.

## Resources

- https://xdebug.org/docs/profiler
- http://kcachegrind.github.io/html/Documentation.html
