---
layout: post
title: "Fix CRLF line separators"
---

Recently, I encountered the following message in PHPStorm:

![You are about to commit CRLF line separators to the Git repository](/images/phpstorm-crlf.png)

> You are about to commit CRLF line separators to the Git repository

The issue arose after adding some XML files to my project, which came from a
Windows computer.

Windows uses CRLF line endings, which stand for "Carriage Return + Line Feed."

- Carriage Return (CR) moves the cursor to the beginning of the line.
- Line Feed (LF) moves the cursor to the next line.

Unlike Windows, Linux performs both actions at once using only LF.

In this article, I'll present multiple strategies to resolve this issue
permanently.

## Configure .editorconfig

The `.editorconfig` file defines the code style for your project. Since it is
versioned, all developers will follow the same conventions. This file should be
placed at the root of your project.

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

Git can also enforce correct line endings using the `.gitattributes` file. Add
the following lines:

```gitignore
# .gitattributes
*.php   eol=lf
*.xml   eol=lf
```

Now, every time a developer pushes or pulls code, Git will automatically apply
the correct line endings.

The `.gitattributes` file can configure many other settings beyond line endings.
For more information, check
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
command to fix, for example, all PHP and XML files. Adapt the command according
to your specific needs:

```console
dos2unix **/*.{php,xml}
```

The `dos2unix` tool offers various features. For full documentation, visit
the [dos2unix homepage](https://dos2unix.sourceforge.io/).

By following these steps, you can prevent and fix CRLF line separator issues in
your project, ensuring consistency across different operating systems.

## Conclusion

Maintaining consistent line endings is crucial when multiple developers work on
a same codebase, with configuration files like `.editorconfig`,
`.gitattributes`, and tools like `dos2unix`, you can effectively fix and prevent
CRLF issues in your project.
