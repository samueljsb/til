---
title: "`fixit fix` makes errors invisible"
date: 2023-11-30
tags:
-   python
-   fixit
---

The following is true as of `fixit` version 2.1.0.

## the problem

[`fixit`](https://pypi.org/project/fixit/) has two commands to lint source code:
`lint` and `fix`. The former will check for errors and correctly exits with a
non-zero exit code when it finds violations of linting rule or errors during
execution. However, the `fix` command will always exit `0` if it does not make
any changes. It will also exit `0` if the linter fails to run at all because of
a problem parsing the source code (no fixes -> exit `0`).

This means that it is never sufficient to only run the `fix` command in CI or
pre-commit hooks. This might be surprising: other linters -- even ones that can
also apply auto-fixes, like [`ruff`](https://pypi.org/project/ruff/) can be
configured to correctly report the errors if they find them but cannot fix them.

## what to do about it

When using `fixit`, make sure you run the `lint` command *as well as* the `fix`
command. For `pre-commit` that means configuring both hooks:

```yaml
repos:
-   repo: https://github.com/Instagram/Fixit
    rev: v2.1.0
    hooks:
    -   id: fixit-fix
    -   id: fixit-lint
```

N.B. The `fix` command is run first so that the `lint` command only reports
remaining errors.

---

I have [reported this issue](https://github.com/Instagram/Fixit/issues/409), so
this may not be true for later version of `fixit`.
