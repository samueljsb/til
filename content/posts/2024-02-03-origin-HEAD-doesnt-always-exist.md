---
title: "`origin/HEAD` doesn't always exist"
date: 2024-02-03
tags:
-   git
---

I always use `main` as the default branch in new repositories,
but at work and elsewhere
I use repositories that still have the old `master` name.
This makes it a bit awkward to write aliases and shell functions
for working with the origin's default branch,
for example when I want to fetch it.

I can [use an alias locally]({{< ref 2023-08-25-git-main-branch-alias >}})
to make muscle memory easier,
but sometimes I need a reference to the *remote* version of the branch.

For that, I have a [shell function in my `.zshrc`](https://github.com/samueljsb/config-files/blob/bbef5d14db577744f3f6e855beb8199603741dcd/templates/.zshrc#L103-L105)
to infer the branch that `origin/HEAD` points to
using `git rev-parse`.

However, this does not always work.
In any repository that started locally and was only later pushed up to a remote,
this `origin/HEAD` reference just doesn't exist
and my function gets an error:

```console
$ git rev-parse --abbrev-ref origin/HEAD
origin/HEAD
fatal: ambiguous argument 'origin/HEAD': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'
```

Repositories that I have cloned from GitHub have this reference,
but when it isn't already present it can be added explicitly:

```console
$ git remote set-head origin -a
origin/HEAD set to main

$ git rev-parse --abbrev-ref origin/HEAD
origin/main
```

---

`origin/HEAD` can also be removed,
if you don't want it hanging around:

```shell
git remote set-head origin -d
```

I don't have any other way to reliably find the remote default branch,
so for now I'm going to leave mine in place.
