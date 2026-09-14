---
layout: post
title: Fix "Invalid US-ASCII character \xE2" error in Asciidoctor
---

When using the official [Asciidoctor Docker image][asciidoctor-image], I
encountered the following error while executing `asciidoctor-epub3`:

```console
Invalid US-ASCII character "\xE2"
  Use --trace to show backtrace
```

This error occurs when the system is misconfigured and defaults to US-ASCII
encoding instead of UTF-8. Even if your text is encoded in UTF-8, you may still
see this error due to incorrect locale settings.

You can resolve this issue using one of two approaches: a comprehensive method
that ensures system-wide locale support, or a quick workaround that forces Ruby
to use UTF-8.

## Check Current Locale

Before applying a fix, check the system's locale settings:

```sh
env | grep LANG
env | grep LC_
```

If the output indicates `LANG=C` or `LANG=POSIX`, your system is likely
defaulting to US-ASCII, which causes encoding issues.

## Long-Term Solution: Configure Locales

To properly configure the system locale, install UTF-8 support. Here is how to
do it in a Debian-based Docker container:

```dockerfile
FROM debian:bookworm

# Install locales and configure system encoding
RUN set -ex \
    && echo "### Install en_US locale ###" \
    && apt-get update && apt-get install -y locales \
    && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && dpkg-reconfigure --frontend=noninteractive locales \
    && update-locale LANG=en_US.UTF-8

# Set environment variables to enforce UTF-8 encoding
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
```

This method ensures the entire system supports UTF-8, preventing similar
encoding issues in other applications.

## Quick Fix: Force Ruby to Use UTF-8

For a simpler solution that fixes Asciidoctor-related errors, force Ruby to use
UTF-8 encoding by setting the `RUBYOPT` environment variable:

```dockerfile
FROM debian:bookworm

# Force Ruby (and Asciidoctor) to use UTF-8 encoding
ENV RUBYOPT -Eutf-8

# The rest of your Dockerfile here...
```

This approach does not modify the system locale but ensures Ruby processes text
as UTF-8, which is often sufficient for resolving `asciidoctor-epub3` encoding
errors.

## Conclusion

The "Invalid US-ASCII character \xE2" error is caused by system locale
misconfiguration, not by your code. If you frequently encounter encoding issues,
it is best to properly configure locales using the long-term solution. However,
if you only need a quick fix for Asciidoctor, the `RUBYOPT` setting may suffice.

Additionally, consider using the official Asciidoctor Docker image if it meets
your requirements, as it comes preconfigured with the correct settings.

[asciidoctor-image]: <https://hub.docker.com/r/asciidoctor/docker-asciidoctor>
