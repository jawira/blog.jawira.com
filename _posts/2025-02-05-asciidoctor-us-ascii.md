---
layout: post
title: "Fixing invalid us-ascii characters \xE2"
---

I had the following issue when using "asciidoctor-epub3" in docker.

````console
Invalid US-ASCII character "\xE2"
  Use --trace to show backtrace
````

If you are sure that you don't have us-ascii encoded text, then the problem si
missonfiguration.

Display current locale.

I have two ways to fix this, the long way and the easy way.

## Long way to fix

Install locales

```dockerfile
FROM debian:bookworm

# https://stackoverflow.com/a/38553499/4345061
RUN set -ex \
    && echo "### Install en_US locale ###" \
    && apt-get --yes --no-install-recommends install locales \
    && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && dpkg-reconfigure --frontend=noninteractive locales \
    && update-locale LANG=en_US.UTF-8
ENV LANG en_US.UTF-8
```

## Easy way to fix

Use RUBYOPT variable to set locale.

```dockerfile
FROM debian:bookworm

# Force Ruby (and Asciidoctor) to use utf8 encoding
ENV RUBYOPT -Eutf-8
```

## Conclusion

As the error comes from a problem with text encoding, however the source of the
problem is system configuration, not your code.
Consider using official docker image, I haven't used because I had to add
special dependencies to my Dockerfile.
