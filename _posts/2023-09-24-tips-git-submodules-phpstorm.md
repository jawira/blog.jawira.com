---
layout: post
title: "PHPStorm tips for working with Git submodules"
---

Submodules are a very handy Git feature, it allows to work with multiple git
repositories at same time. In practice this means that I only need one PHPStorm
windows to work with two or more projects.

In this article I share some useful tips to ease the process of working with
submodules.

## Directory structure

First, submodules are meant to be used with related projects, the most common
use case is a project with a front-end and a back-end, both are needed in order
to develop a new feature.

Imagine two repositories, `foo-backend` and `foo-frontend`. In order to work
with these two repositories create a new repository, for example `foo-stack`,
this project will contain `foo-backend` and `foo-frontend` as submodules.

```console
$ git clone git@github.com:jawira/foo-stack.git
$ cd foo-stack
$ git submodule add git@github.com:jawira/foo-backend.git
$ git submodule add git@github.com:jawira/foo-frontend.git
```

> Note: I'm using SSH to clone repositories, but submodules works also with
> HTTPS.
> {:.info}

## Cloning project with submodules

To clone a project containing submodules you have to use two extra options.

```console
$ git clone --recurse-submodules --remote-submodules git@github.com:jawira/foo-stack.git
```

* `--recurse-submodules`: Clone project's submodules.
* `--remote-submodules`: Fetch submodules to latests reference.

## Submodules branches

Working with submodules means working with multiple repositories at the same
time, in consequence it also means working with multiple branches at same time.
Depending on your project, this can be overwhelming.

Instead of git commands, I recommend to use PHPStorm GUI to commit, push, and
switching between branches.

Additionally, **I strongly recommend** the usage
of [Git Extender](https://plugins.jetbrains.com/plugin/7835-git-extender),
this PHPStorm plugin will allow you to update ALL your branches (including
submodules branches) with a simple shortcut <kbd>ctrl+t</kbd>, optionally it can
also delete local branches after a merge.

## Conclusion

Working with submodules can make you save a lot of time, on the other hand,
without the right tooling and knowledge working with submodules can be very
frustrating.
