---
layout: post
title: "PhpStorm tips for working with Git Submodules"
---

Submodules are a very handy Git feature. They allow you to work with multiple
Git repositories at the same time. In practice, this means that I only need one
PhpStorm window to work with two or more projects.

In this article, I share some useful tips for working with submodules.

## Directory structure

First, submodules are meant to be used with related projects. The most common
use case is a project with a front-end and a back-end; both are needed to
develop a new feature.

Imagine two repositories, `foo-backend` and `foo-frontend`. To work with these
two repositories, create a new repository, for example, `foo-stack`. This
project will contain `foo-backend` and `foo-frontend` as submodules.

```console
$ git clone git@github.com:jawira/foo-stack.git
$ cd foo-stack
$ git submodule add git@github.com:jawira/foo-backend.git
$ git submodule add git@github.com:jawira/foo-frontend.git
```

> **Note:** I'm using SSH to clone repositories, but submodules also work with
> HTTPS.
> {:.info}

## Cloning project with submodules

To clone a project containing submodules, you must use two extra options:

```console
$ git clone --recurse-submodules --remote-submodules git@github.com:jawira/foo-stack.git
```

* `--recurse-submodules`: Clone the project's submodules.
* `--remote-submodules`: Fetch submodules to the latest reference.

## Submodule branches

Working with submodules means working with multiple repositories at the same
time. In consequence, it also means working with multiple branches at the same
time. Depending on your project, this can be overwhelming.

Instead of Git commands, I recommend using the PhpStorm GUI to commit, push, and
switch between branches.

Additionally, **I strongly recommend**
using [Git Extender](https://plugins.jetbrains.com/plugin/7835-git-extender).
This PhpStorm plugin allows you to update all your branches (including submodule
branches) with a simple shortcut <kbd>ctrl+t</kbd>. Optionally, it can also
delete local branches after a merge.

## Conclusion

Working with submodules can save you a lot of time. On the other hand, without
the right tooling and knowledge, working with submodules can be very
frustrating.
