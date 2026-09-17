---
layout: post
title: "Using SSH tunnels for Docker databases"
---

Most local Docker stacks have a database in them. To inspect it from an IDE or
database client, the usual move is to publish its port on the host.

It is convenient, but it has two drawback:

1. The published ports can conflict between projects.
2. A published port can become accessible on all the host's network interfaces.

This article proposes an alternative to _port mapping_: a single SSH entry point
that lets database clients reach containers on a shared Docker network. It also
covers how to make conventional _port mapping_ safer when an SSH tunnel is more
complexity than you need.

## The usual approach

Publishing a container port makes the database available to tools on the host.
Here is a minimal PostgreSQL Compose file:

```yaml
# rebel/compose.yaml
services:

  rebel-database:
    image: postgres:18
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: anakin
      POSTGRES_PASSWORD: T3chn0Uni0n
      POSTGRES_DB: jedi
```

`ports` maps the container's port `5432` to port `5432` on the host. PhpStorm,
pgAdmin, and any other clients can then connect to it directly.

_Port mapping_ is very convenient, but it has two drawbacks:

* A port can be used by only one process on the host, a second PostgreSQL
  project cannot also use the port `5432`. This is called _port collision_.
* By default, Docker publishes the port on all host interfaces. This make a
  development database reachable by other machines on all host's networks.

The next section explains a replaces the per-database mappings with an SSH
tunnel. Later, we will return to the simpler approach and show how to limit a
published port to the local machine.

## The tunnel approach

This solution is composed of two parts:

1. Creating and configuring an OpenSSH server, this container acts as the single
   entry point for database clients.
2. Configuring existing projects to use the SSH tunnel instead of using port
   mapping.

### Create the tunnel project

Create a small project named `tunnel` with one `compose.yaml` file:

```text
tunnel/
└── compose.yaml
```

This uses the LinuxServer OpenSSH image with its SSH-tunnel mod:

```yaml
# tunnel/compose.yaml
services:

  ssh:
    image: lscr.io/linuxserver/openssh-server:latest
    restart: unless-stopped
    environment:
      DOCKER_MODS: linuxserver/mods:openssh-server-ssh-tunnel
      USER_NAME: tunneluser
      PUBLIC_KEY_FILE: /tmp/tunnel_ssh.pub
    volumes:
      - ~/.ssh/tunnel_ssh.pub:/tmp/tunnel_ssh.pub:ro
    ports:
      - "127.0.0.1:2222:2222"
```

The relevant bits are:

* `USER_NAME` is the SSH user used by the database client.
* `PUBLIC_KEY_FILE` is the public-key path **inside** the container. It must
  match the target of the bind mount.
* Port `2222` is the only port published to the host, and it is bound to the
  loopback interface. Database ports remain internal to Docker.

Compose creates a default network automatically. For a project named `tunnel`,
that network is `tunnel_default`. Other Compose projects can attach to it as an
external network.

Start the SSH server:

```console
docker compose up -d
```

The companion [tunnel repository](https://github.com/jawira/tunnel) has a
ready-to-use OpenSSH configuration and a Makefile that generates a key pair and
starts the container.

### Connect a database project to the tunnel network

Now let's see how to configure an existing project in order to use the SSH
tunnel. Here, that project is `rebel`:

```text
rebel/
└── compose.yaml
```

This is the updated `compose.yaml`:

```yaml
# rebel/compose.yaml
services:

  rebel-database:
    image: postgres:18
    environment:
      POSTGRES_USER: anakin
      POSTGRES_PASSWORD: T3chn0Uni0n
      POSTGRES_DB: jedi
    networks:
      tunnel_default:

networks:
  tunnel_default:
    external: true
```

As you can see, there is no `ports` section. The database is available only to
containers connected to the `tunnel_default` network.

Docker's internal DNS resolves `rebel-database` to this container. Every
container attached to the shared network needs a unique service name. That name
becomes a hostname on the network, so duplicate names create a collision.

Finally, start the project as usual:

```console
docker compose up -d
```

### Configuring PhpStorm

This section explains how to configure PhpStorm in order to reach the database
over the SSH tunnel we just created.

The first step is to create a PostgreSQL data source in the Database tool
window.

![Database tool window](/images/tunnel-1.png)

On the **General** tab, enter the credentials from the `rebel` Compose file:

* Host: `rebel-database`
* Port: `5432` (PostgreSQL default port)
* User: `anakin`
* Password: `T3chn0Uni0n`
* Database: `jedi`

![Database General tab](/images/tunnel-2.png)

Then open the **SSH/SSL** tab, enable **Use SSH tunnel**, and create an SSH
configuration.

![SSH SSL tab](/images/tunnel-3.png)

Use the SSH credentials from the `tunnel` project:

* Host: `localhost`
* Port: `2222`
* Username: `tunneluser`
* Authentication type: **Key pair**
* Private key file: `~/.ssh/tunnel_ssh`

![Tunnel credentials](/images/tunnel-4.png)

Return to the **General** tab and select **Test Connection**.

![Test database connection](/images/tunnel-5.png)

PhpStorm now reaches the database hostname through the SSH connection.

![Database connection](/images/tunnel-6.png)

Other databases can reuse this SSH configuration. They only need to join
`tunnel_default` and use distinct service names.

## Improving the port mapping approach

An SSH tunnel gives you one entry point and avoids assigning a host port to
every database. It also adds a project, network, key pair, and IDE setup. For
many teams, ordinary port publishing is still the more practical choice.

Two improvements can be done to the classic approach.

First, in order to avoid port collisions, give each project its own host port
and write the allocation down. For example:

| Project                  | Port mapping |
|--------------------------|--------------|
| jawira/turbo-eureka      | 54320:5432   |
| jawira/miniature-dollop  | 54321:5432   |
| jawira/literate-disco    | 54322:5432   |
| jawira/improved-broccoli | 54323:5432   |

A small port list in the team wiki is low-tech, but easy to understand and works
well while the number of local projects is manageable.

Secondly, when you publish a database port, bind it explicitly to the loopback
interface:

```yaml
services:
  db:
    ports:
      - "127.0.0.1:5432:5432"
```

Unlike `5432:5432`, this mapping is available only from the Docker host. It
remains accessible to local tools but is not exposed on the host's network
interfaces.

## Conclusion

There's no perfect solution.

SSH tunneling is an "elegant" solution, it replaces database _port mapping_ on
the host with one single entry point. You no longer need a separate port for
every database.

However, each service name needs to be unique, meaning that you replaced the
risk of port collision with the risk of hostname collision.

If your team uses a Wiki, documenting port allocation, and binding each database
port to `127.0.0.1` is often the better trade-off.

## A few notes

* The setup presented in this post is intended for local development. Do not use
  it in production without reviewing authentication, network exposure, secrets
  management, and operational requirements.
* The complete tunnel project is available
  at <https://github.com/jawira/tunnel>; its Makefile can generate a key pair
  and start the SSH container.
