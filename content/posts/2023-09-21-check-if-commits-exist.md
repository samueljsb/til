---
title: How to check whether a commit exists
date: 2023-09-21
tags:
-   git
---

## ignoring revisions with `git blame`

[`git blame`] is a useful tool for understanding how code came to be the way it
is, especially in a codebase that has many authors or has been around for a long
time. However, not all commits are equal: "run black on all files" might touch
every single file, but the change isn't particularly helpful for understanding
the code itself. Because of this, `git` allows you to ignore a particular
revision when blaming a file (`--ignore-rev`). You can also ignore a whole batch
of revisions by keeping them in a file (`--ignore-revs-file`). GitHub will
[automatically use this option] in the blame view if there is a
`.git-blame-ignore-revs` file in the root of a repository.

[`git blame`]: https://www.git-scm.com/docs/git-blame
[automatically use this option]:
    https://docs.github.com/en/repositories/working-with-files/using-files/viewing-a-file#ignore-commits-in-the-blame-view

## a common problem

We use a lot of automated tools to lint and format our code at work and we
sometimes add new standards or tool which cause large reformats that don't
change any functionality. So we keep a `.git-blame-ignore-revs` file to keep
track of which commits aren't useful to blame. But is every entry in that file a
real commit?

There's a common problem that arises when we add commit SHAs to this file: when
a branch is rebased on `main`, before merging, the commit SHAs all change. A
common process looks like:

1. Run the tool and commit the changes (SHA: `abc123`)
2. Add the refactor commit SHA to the blame ignore file
3. Push the code up to be reviewed
4. Once the code has been approved, rebase on `main` to make sure we don't have
   implicit conflicts (new SHA: `def456`)
5. Merge

This means we have a refactor commit with the SHA `def456`, but we have saved
`abc123` in our ignore file. This problem tends to go unnoticed because lines
in that file that don't match a commit are just ignored.

## the solution

So how do we check whether these commits are real?

```sh
git cat-file -e $SHA
```

This will exit with a non-zero code (1) if the SHA is not a commit in this
repository. To check the contents of a whole blame ignore file and print the
invalid lines:

```sh
xargs -a .git-blame-ignore-revs --replace bash -c 'git cat-file -e {} || echo {}'
```

N.B. This is using the GNU `xargs` -- the replace behaviour is slightly
different in the BSD version included with macOS.
