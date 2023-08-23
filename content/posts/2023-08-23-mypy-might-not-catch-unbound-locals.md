---
title: mypy won't always catch unbound locals
date: 2023-08-23
tags:
-   python
-   mypy
---

It is possible to write code with an unbound local bug that `mypy` does not
catch.

## undefined names

It is possible to write code that cannot be executed because it tries to refer
to names that have not been defined:

```python
x = 3

print(y)
```

This will raise an error:

```pytb
Traceback (most recent call last):
  File "<frozen runpy>", line 198, in _run_module_as_main
  File "<frozen runpy>", line 88, in _run_code
  File "/private/tmp/tmpvenv-48a7d/t.py", line 3, in <module>
    print(y)
          ^
NameError: name 'y' is not defined
```

And `mypy` catches this bug:

```console
$ mypy t.py
t.py:3: error: Name "y" is not defined  [name-defined]
Found 1 error in 1 file (checked 1 source file)
```

This usually happens when a name is misspelled or an import is forgotten.

## where it goes wrong

Another common place for undefined names is in non-exhaustive `if`/`else`
statements that assign that name in each clause. Unfortunately, `mypy` does not
catch this problem:

```python
def f(x: int) -> None:
    if x == 1:
        msg = "one"
    elif x == 2:
        msg = "two"

    print(msg)

f(3)
```

```console
$ mypy t.py
Success: no issues found in 1 source file
```

And yet when we try to run this, we get an `UnboundLocalError`:

```pytb
Traceback (most recent call last):
  File "<frozen runpy>", line 198, in _run_module_as_main
  File "<frozen runpy>", line 88, in _run_code
  File "/private/tmp/tmpvenv-48a7d/t.py", line 9, in <module>
    f(3)
  File "/private/tmp/tmpvenv-48a7d/t.py", line 7, in f
    print(msg)
          ^^^
UnboundLocalError: cannot access local variable 'msg' where it is not associated with a value
```
