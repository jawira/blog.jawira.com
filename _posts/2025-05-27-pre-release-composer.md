---
layout: post
title: 'Updating caret constraints for "^v0" versions with Composer'
---

Composer has special behavior when handling `v0` versions with caret
constraints.

Any `v0.x.y` version is considered a pre-release. These versions are unstable
and can introduce backward compatibility (BC) breaks at any time.

Using the caret (`^`) with pre-releases has special behavior in Composer, and
this is by design.

In essence, they only update within patch versions. This is designed to protect
you from breaking changes, but it is poorly explained in the Composer
documentation:

> For pre-1.0 versions it also acts with safety in mind and treats ^0.3
> as >=0.3.0 <0.4.0 and ^0.0.3 as >=0.0.3 <0.0.4.

![Composer documentation showing caret pre-release behavior](/images/caret-pre-release.png)

Source: <https://getcomposer.org/doc/articles/versions.md#caret-version-range->

For example:

| Constraint | Versions          |
|------------|-------------------|
| ^1.2.0     | >=1.2.0 <2.0.0    |
| ^5.0.4     | >=5.0.4 <6.0.0    |
| ^13.5      | >=13.5.0 <14.0.0  |
| ^0.3.0     | >=0.3.0 <0.4.0 ⚠️ |

This behavior is very counterintuitive.

If the software you are working on is stable, I recommend migrating to `v1`
releases as soon as possible.
