---
title: Python modules can import themselves
date: 2024-11-14
tags:
-   python
---

A python module can import itself.
Whilst this sounds, on its face, paradoxical,
Python handles it gracefully.
It does not crash, nor recurse infinitely,
but it allows for some fun and unexpected behaviour.

## a simple example

Consider a simple package in `/tmp/pkg/`,
containing a single module, `a.py`:

```python
import pkg.a

print(f"in {__file__}")
```

When imported from another module,
this will print `in /tmp/pkg/a.py`:

```pycon
>>> import pkg.a
in /tmp/pkg/a.py
```

However, when the module itself is run:

```console
$ cd /tmp
$ python3 -m pkg.a
in /tmp/pkg/a.py
in /tmp/pkg/a.py
```

It prints twice!
But only twice?

It appears that the first time the interpreter encounters `import pkg.a`,
it follows the import and starts executing `pkg.a` (again).
But when it encounters that line for the second time,
it somehow knows that `pkg.a` has already been found,
so does not recurse further.

To understand why this happens, we can change the `print` statement
to show us which module is being executed:

```diff
- print(f"in {__file__}")
+ print(f"{__name__} in {__file__}")
```

Now we get:

```console
$ python3 -m pkg.a
pkg.a in /tmp/pkg/a.py
__main__ in /tmp/pkg/a.py
```

The first time the interpreter encounters the `import` statement,
it is executing the module `__main__`.
Therefore, it loads and executes the appropriate file to load module `pkg.a`.
The second time it encounters that statement,
it is already in `pkg.a`.
Python doesn't waste time executing the same modules over and over again
wherever they are imported.
If the module already exists in `sys.modules`,
that will be used instead of a new import.

In this case, `pkg.a` is not in `sys.modules` when the interpreter is executing `__main__`.
But it is there once we start importing `pkg.a` the first time.
This prevents an infinite recursion.

## unexpected execution order

We can start to do all sorts of fun things with this.

I started playing around with this when I posed this riddle to my colleagues.
If we have a package in `/tmp/pkg/`,
containing a two modules,
`a.py`:

```python
import pkg.a

print("in a")

import pkg.b
```

and `b.py`:

```python
print("in b")
```

What happens when we run `python3 -m pkg.a`?

The result is, at first, surprising:

```console
$ python3 -m pkg.a
in a
in b
in a
```

I'll leave it as an exercise to the reader
to explain why this happens
(the previous section should offer the clues).

## duplicate singletons

Sometimes, we create an object in a module at import time, intending that to be
the only instance of the object: a singleton. However, this self-import trick
allows us to *double* the number of singletons we have.

If I have `pkg/a.py` containing:

```python
import pkg.a

singleton = object()
```

One would expect,
just by reading the code,
that there would only be a single object created.
However, loading the module in an interactive interpreter shows *two*:

```console
$ python3 -im pkg.a
>>> singleton
<object object at 0x1021e86a0>
>>> pkg.a.singleton
<object object at 0x1021e8690>
>>> singleton is pkg.a.singleton
False
```

We can also use this to show classes being defined twice!
When we subclass `float`
(because it has no subclasses in the standard library,
so there are no distractions in the example):

```python
import pkg.a

class MyClass(float):
    pass
```

We can see two independent subclasses:

```console
$ python3 -im pkg.a
>>> float.__subclasses__()
[<class 'pkg.a.MyClass'>, <class '__main__.MyClass'>]
```

## self-referential echoes

If, instead of writing `import pkg.a`,
we assign the imported module to an alias:
`import pkg.a as a`,
or if we use the `from pkg import a` format,
we can reference the module from itself.

For example,
if we alias the import in the singleton example from before:

```python
import pkg.a as a

singleton = object()
```

We can now reference the imported module as `a`,
or `a.a`, or `a.a.a`, or `a.a.a.a`...

```console
$ python3 -im pkg.a
>>> singleton
<object object at 0x1005b86a0>
>>> a.a.a.a.a.a.a.a.a.a.a.singleton
<object object at 0x1005b8690>
```

This is, of course, extremely silly.
