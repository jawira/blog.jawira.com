---
layout: post
title: "Fix '502 Bad Gateway' with Xdebug 3"
---

If you've encountered issues with Xdebug while working on a Symfony project
(particularly when PhpStorm is listening for Xdebug), you might have come across
a "502 Bad Gateway" error from the Nginx server. In this post, I'll share how I
resolved this problem.

## My environment

I'm currently working on a Symfony 6.1 application in a Dockerized environment:

- Nginx 1.19
- PHP 8.1 FPM

Upon attempting to listen for Xdebug in PhpStorm, I encountered the following
error in the FPM container:

```
[05-Mar-2024 09:56:20] WARNING: [pool www] child 9 exited on signal 11 (SIGSEGV - core dumped) after 10.753537 seconds from start,
```

Simultaneously, the browser displayed the following error:

![Nginx 502 Bad Gateway error](/images/nginx-502-bad-gateway.png)

## The solution

After several hours of investigation, I pinpointed the root cause of the issue
and discovered a solution. It turns out there is a bug in Xdebug 3.3.*,
documented in various tickets:

- [Issue #2229](https://bugs.xdebug.org/view.php?id=2229)
- [Issue #2235](https://bugs.xdebug.org/view.php?id=2235)
- [Issue #2244](https://bugs.xdebug.org/view.php?id=2244)

Since I'm working in a Dockerized environment, I needed to downgrade Xdebug
within the Docker container, not on the host system.

Here is my Dockerfile after the downgrade:

```dockerfile
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

Note that you need to specify the Xdebug version as `3.2.2`. Remember, you need
to rebuild and restart your container for the changes to take effect.

## Conclusion

The root cause of the problem is a bug in Xdebug 3.3.*. Downgrading to Xdebug
3.2.2 successfully resolved the issue.
