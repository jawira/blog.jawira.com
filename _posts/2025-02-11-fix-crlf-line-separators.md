---
layout: post
title: "Fix CRLF line separators"
---

Recently, I encountered the following message in PhpStorm:

![You are about to commit CRLF line separators to the Git repository](/images/phpstorm-crlf.png)

> You are about to commit CRLF line separators to the Git repository

The issue arose after adding some XML files to my project that came from a
Windows computer.

Windows uses CRLF line endings, which stand for "Carriage Return + Line Feed":

- Carriage Return (CR) moves the cursor to the beginning of the line.
- Line Feed (LF) moves the cursor to the next line.

Unlike Windows, Linux performs both actions at once using only LF.

In this article, I'll present multiple strategies to resolve this issue
permanently.

## Configure .editorconfig

The `.editorconfig` file defines the code style for your project. Because it is
version-controlled, all developers will follow the same conventions. This file
should be placed at the root of your project.

To enforce LF line endings, add the following configuration:

```editorconfig
# .editorconfig
[*.{php,xml}]
end_of_line = lf
```

In this example, we specify that PHP and XML files should use LF line endings.
Adapt this setting based on your project requirements.

For more details, visit the [EditorConfig homepage](https://editorconfig.org/).

## Configure .gitattributes

Git can also enforce correct line endings using the `.gitattributes` file.

In the following example, we specify the line ending for PHP and XML files:

```gitattributes
# .gitattributes
*.php text eol=lf
*.xml text eol=lf
```

You can also define the line ending for all text files with a single line:

```gitattributes
# .gitattributes
* text=auto eol=lf
```

Here is a full example:

```gitattributes
# .gitattributes

# Linux
* text=auto eol=lf

# Windows
*.bat   text eol=crlf
*.cmd   text eol=crlf

# Binary
*.phar  binary
*.png   binary
```

Once you have created the `.gitattributes` file, you have to execute the
following command once to update the line endings of all existing files:

```console
git add --renormalize .
```

From now on, Git will automatically apply the correct line endings whenever
developers push or pull code.

This file can also configure many other settings beyond line endings. For more
information, check
the [Git Attributes documentation](https://git-scm.com/docs/gitattributes).

## Replacing all CRLF occurrences with LF

So far, we've covered preventive measures to handle CRLF issues. But what about
existing files?

To change line endings in existing files, use the `dos2unix` tool. First,
install it using `apt`:

```console
apt install dos2unix
```

Once installed, navigate to the root of your project and run the following
command to fix all PHP and XML files, for example. Adapt the command according
to your specific needs:

```console
dos2unix **/*.{php,xml}
```

The `dos2unix` tool offers various other features. For full documentation, visit
the [dos2unix homepage](https://dos2unix.sourceforge.io/).

By following these steps, you can prevent and fix CRLF line separator issues in
your project, ensuring consistency across different operating systems.

## Conclusion

Maintaining consistent line endings is crucial when multiple developers work on
the same codebase. With configuration files like `.editorconfig`,
`.gitattributes`, and tools like `dos2unix`, you can effectively fix and prevent
CRLF issues in your project.
