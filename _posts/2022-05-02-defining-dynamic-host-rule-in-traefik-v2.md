---
layout: post
title: "Defining dynamic host rules in Traefik v2"
---

[Traefik](https://traefik.io/traefik/) is a reverse HTTP proxy. It works
especially well with [Docker](https://www.docker.com/)
and [Docker Compose](https://docs.docker.com/compose/) environments. Traefik
receives all incoming web requests and redirects them to Docker containers.

<!-- {% raw %} -->
![Traefik diagram](/images/traefik_diagram.png)

To redirect requests, Traefik uses a router to identify the appropriate web
server.

In this article, I'll show you how to configure a default host rule for a
development environment. This rule will automatically apply to any service:

- **`http://<service>.<project>.localhost`**

This article assumes you have some experience with Docker and Traefik.

## Traefik configuration

First, install Traefik. Let's create a new project with the following structure:

```
traefik/
└── compose.yaml
```

You only need one file to start using Traefik. Here is the content of
`traefik/compose.yaml`:

```yaml
services:

  traefik:
    image: traefik:v2.7
    restart: always
    command:
      - '--api.insecure=true'
      - '--providers.docker.exposedByDefault=false'
      - '--providers.docker.network=traefik_default'
      - '--providers.docker.defaultRule=Host(`{{ index .Labels "com.docker.compose.service" }}.{{ index .Labels "com.docker.compose.project" }}.localhost`)'
    labels:
      - 'traefik.http.services.traefik-traefik.loadBalancer.server.port=8080'
      - 'traefik.enable=true'
    ports:
      - '80:80'
      - '8080:8080'
    volumes:
      - '/var/run/docker.sock:/var/run/docker.sock'
    networks:
      - default

networks:
  default:
```

The most important command option for creating dynamic rules is:

```yaml
- '--providers.docker.defaultRule=Host(`{{ index .Labels "com.docker.compose.service" }}.{{ index .Labels "com.docker.compose.project" }}.localhost`)'
```

Here's what this command does:

- **`--providers.docker.defaultRule`**: This is the rule Traefik uses if the
  container doesn't define one.
- **`Host()`**: Traefik uses the requested domain for routing.
- **`{{ index .Labels "com.docker.compose.service" }}`**: This represents the
  service name that you define in `compose.yaml`.
- **`{{ index .Labels "com.docker.compose.project" }}`**: This placeholder is
  replaced with the project name. By default, the project name is the directory
  name. You can also specify this value using the `-p <project>` option.
- Finally, **`.localhost`**
  is [a reserved domain](https://datatracker.ietf.org/doc/html/rfc2606). It will
  always point to the loopback address **`127.0.0.1`**. Using `.localhost`, you
  do not need to add your development domains to `/etc/hosts`.

Start Traefik with the following command:

```console
$ docker compose -p traefik up -d
```

Traefik is now up and running.

## Creating a sample project

Let's create a demo project to verify our dynamic rules work correctly. This is
the structure of the `foo` project:

```
foo/
└── compose.yaml
```

Let's define an Apache server in `foo/compose.yaml`:

```yaml
services:

  web:
    image: httpd
    labels:
      - traefik.enable=true
    networks:
      - default
      - traefik_default

networks:
  default:
  traefik_default:
    external: true
```

Let's run our demo project:

```console
$ docker compose -p foo up -d
```

Now, if we open `http://web.foo.localhost`, we will see Apache's welcome
message:

![Traefik Apache default rule works](/images/traefik_apache_works_1.png)

## Using a custom rule

We can also specify a custom rule for a service, which will override the default
rule.

For example, to use `my-foo.localhost`, we simply add a new label:

```diff
# ...
  web:
    image: httpd
    labels:
      - traefik.enable=true
+     - traefik.http.routers.foo-project.rule=Host(`my-foo.localhost`)
# ...
```

To apply the changes, we need to restart the project using
`docker compose -p foo up -d`. Here is the result:

![Traefik Apache custom rule works](/images/traefik_apache_works_2.png)

As you can see, everything works as expected.

## Conclusion

Traefik is a highly configurable reverse proxy. You only need one line of code
to create dynamic domains, which are well suited for development environments.

If you use the `.localhost` TLD, you won't need to declare your domains in
`/etc/hosts`.
<!-- {% endraw %} -->
