---
layout: post
title: "Fixing Invalid US-ASCII Characters in Asciidoctor"
---

## Introduction

While using the official [Asciidoctor Docker image][asciidoctor-image],
I encountered the following error when I executed `asciidoctor-epub3`:

```console
Invalid US-ASCII character "\xE2"
  Use --trace to show backtrace
```

This error occurs when the system is misconfigured and defaults to US-ASCII
encoding instead of UTF-8. If your text is encoded in UTF-8 and you still see
this error, it is likely due to incorrect locale settings.

To resolve this issue, there are two approaches: a comprehensive method that
ensures system-wide locale support and a quick workaround that forces Ruby to
use UTF-8.

## Display Current Locale

Before applying a fix, check your system’s locale settings using:

```sh
env | grep LANG
env | grep LC_
```

If the output indicates `LANG=C` or `LANG=POSIX`, your system is likely
defaulting to US-ASCII, which causes encoding issues.

## Long-Term Solution: Installing Locales

To properly configure the system’s locale, install and set up UTF-8 support.
Here’s how to do it in a Debian-based Docker container:

```dockerfile
FROM debian:bookworm

# Install locales and configure system encoding
RUN set -ex \
    && echo "### Install en_US locale ###" \
    && apt-get update && apt-get install -y locales \
    && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && dpkg-reconfigure --frontend=noninteractive locales \
    && update-locale LANG=en_US.UTF-8

# Set environment variable to enforce UTF-8 encoding
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
```

This method ensures that the entire system supports UTF-8, preventing similar
encoding issues in other applications.

## Quick Fix: Forcing Ruby to Use UTF-8

If you want a simpler solution and only need to fix Asciidoctor-related errors,
you can force Ruby to use UTF-8 encoding by setting the `RUBYOPT` environment
variable:

```dockerfile
FROM debian:bookworm

# Force Ruby (and Asciidoctor) to use UTF-8 encoding
ENV RUBYOPT -Eutf-8

# The rest of your Dockerfile here...
```

This approach does not modify the system’s locale but ensures that Ruby
processes text as UTF-8, which is often sufficient for resolving
`asciidoctor-epub3` encoding errors.

## Conclusion

The "Invalid US-ASCII character \xE2" error stems from system locale
misconfiguration rather than an issue with your code. If you frequently
encounter encoding issues, it’s best to properly configure locales as shown in
the long-term solution. However, if you only need a quick fix for Asciidoctor,
setting `RUBYOPT` may be enough.

Additionally, consider using the official Asciidoctor Docker image if it meets
your requirements, as it comes preconfigured with the correct settings.

[asciidoctor-image]: <https://hub.docker.com/r/asciidoctor/docker-asciidoctor>
