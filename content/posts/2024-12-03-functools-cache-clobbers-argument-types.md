---
title: "`functools.cache` clobbers argument types"
date: 2024-12-03
tags:
-   python
-   mypy
---

`functools.cache` preserves return type annotations,
but clobbers the argument types.
This can lead to code that passes a type-checker,
but contains an obvious bug.

Consider:

```python
import functools
from typing import reveal_type

def func(a: int, b: str) -> int:
    return 0

reveal_type(func)
func(1, 'x')  # OK
func(1, 2)  # error: Argument 2 to "f" has incompatible type "int"; expected "str"

cached_func = functools.cache(func)
reveal_type(cached_func)
cached_func(1, 'x')  # OK
cached_func(1, 2)  # error?
```

That last line should be an error,
but when we run `mypy`:

```console
$ mypy t.py
t.py:7: note: Revealed type is "def (a: builtins.int, b: builtins.str) -> builtins.int"
t.py:9: error: Argument 2 to "func" has incompatible type "int"; expected "str"  [arg-type]
t.py:12: note: Revealed type is "functools._lru_cache_wrapper[builtins.int]"
Found 1 error in 1 file (checked 1 source file)
```

We only see one error!
The bug on the last line is not reported at all.

The revealed type hints at what is happening here:
the return type annotation is being preserved,
but not the arguments.
We can see that in [the annotation](https://github.com/python/mypy/blob/411e1f168e197dd7ca08c820ae04b3160f41071b/mypy/typeshed/stdlib/functools.pyi#L202),
the arguments are ignored.

h/t to my colleague [Charlie Denton](https:/meshy.co.uk)
for finding the annotations in the typeshed.
