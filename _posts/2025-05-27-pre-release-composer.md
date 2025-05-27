---
layout: post
title: "Updating ^v0 constraints with Composer"
---

**_Composer_ has a special behavior when handling `v0` versions with caret
constraints.**

Any `v0.x.y` version is considered a pre-release. These versions are
**unstable** and can introduce BC (backward compatibility) breaks at any time.

Using the caret (`^`) with pre-releases has special behavior in Composer — and
yes, this is by design.<br>
Basically, **they don't update beyond patch versions**.
It's meant to protect you from breaking changes, but it's explained pretty
poorly in _Composer_ docs:

> For pre-1.0 versions it also acts with safety in mind and treats ^0.3
> as >=0.3.0 <0.4.0 and ^0.0.3 as >=0.0.3 <0.0.4.

![Composer documentation](/images/caret-pre-release.png)

Source: <https://getcomposer.org/doc/articles/versions.md#caret-version-range->

For example:

| Constraint | Versions          |
|------------|-------------------|
| ^1.2.0     | >=1.2.0 <2.0.0    |
| ^5.0.4     | >=5.0.4 <6.0.0    |
| ^13.5      | >=13.5.0 <14.0.0  |
| ^0.3.0     | >=0.3.0 <0.4.0 ⚠️ |

This is very counterintuitive.

If the software you are working on is stable, I suggest jumping to `v1` releases
asap.
