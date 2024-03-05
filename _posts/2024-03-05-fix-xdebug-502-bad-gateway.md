---
layout: post
title: "Fix '502 Bad Gateway' with Xdebug 3"
---

If you've encountered issues with Xdebug while working on a Symfony project
( particularly when PHPStorm is listening for Xdebug), you might have come
across a "502 Bad Gateway" error from Nginx server. In this post, I'll share how
I resolved this problem.

## My environment

I'm currently working on a Symfony 6.1 application within a dockerized
environment:

* nginx 1.19
* php 8.1 fpm

Upon attempting to listen for Xdebug in PHPStorm, I encountered the following
error in the FPM container:

```
[05-Mar-2024 09:56:20] WARNING: [pool www] child 9 exited on signal 11 (SIGSEGV - core dumped) after 10.753537 seconds from start,
```

Simultaneously, the browser displayed the following error:

![ngnix error](/images/nginx-502-bad-gateway.png)

## The solution

After several hours of investigation, I pinpointed the root cause of the issue
and discovered a solution. It turns out that there's a bug within Xdebug 3.3.*,
which has been documented in various tickets:

* [Issue #2229](https://bugs.xdebug.org/view.php?id=2229)
* [Issue #2235](https://bugs.xdebug.org/view.php?id=2235)
* [Issue #2244](https://bugs.xdebug.org/view.php?id=2244)

Given that I'm working in a dockerized environment, I needed to downgrade Xdebug
within the Docker container, not on the host system.

Here's what my Dockerfile looked like after the downgrade:

```Dockerfile
# ...

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/bin/install-php-extensions
RUN install-php-extensions \
        pdo \
        pdo_pgsql \
        pgsql \
        xdebug-3.2.2 \
        xsl \
        zip

# ...
```

Note how you specify the Xdebug version to use to `3.2.2`.
Remember, you'll need to rebuild and restart your container for the changes to
be applied.

## Conclusion

The root cause of the problem lies in a bug within Xdebug 3.3.*. Downgrading to
Xdebug 3.2.2 effectively resolved the issue.
