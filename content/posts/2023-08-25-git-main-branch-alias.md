---
title: Alias the default git branch to `main`
date: 2023-08-25
tags:
-   git
---

The name of the default branch in most git repos I work in is `main`. But the
default branch in our primary work repo is still `master` (it predates the
change in conventional name and changing it is a big enough job that nobody has
found time for it yet).

That can be annoying, for scripting and muscle-memory reasons, but it can be
remapped locally:

```sh
git switch -c main --track origin/master
git branch -d master
```

This creates a local branch called `main` that pulls from the remote branch
called `master`.

h/t to my colleague [Daisy Brenecki](https://daisy.wtf) for pointing this out.
