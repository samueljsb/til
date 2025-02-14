---
title: "`typing.runtime_checkable` does not check attribute types"
date: 2025-02-10
tags:
-   python
-   mypy
---

Python has the concept of "structural subtyping",
which allows a type checker (e.g. `mypy` or `pyright`)
to verify the *shape* of something
without requiring it to be an instance of a particular type
(or subtype).

One way to tell the type checker what shape we want something to be
is to write a class that inherits from `typing.Protocol`,
introduced by [PEP 544](https://peps.python.org/pep-0544/).
Protocols usually cannot be used with `isinstance` or `issubclass` checks,
so we can't verify them at runtime.
If we try, we get an error:

```pytb
TypeError: Instance and class checks can only be used with @runtime_checkable protocols
```

Those checks can be allowed
by decorating the protocol with `typing.runtime_checkable`.
However, that can be misleading!

```python
>>> import dataclasses
>>> from typing import Protocol
>>> from typing import runtime_checkable
>>> @runtime_checkable
... class HasXInt(Protocol):
...     x: int
...
>>> @dataclasses.dataclass
... class HasXStr:
...     x: str
...
>>> obj = HasXStr('...')
>>> isinstance(obj, HasXInt)
True
```

This tells us that `c` is an instance of `MyProto`,
even though `c.x` is a string and we expected an integer!
That's because `typing.runtime_checkable`
only checks for the presence of attributes,
not for their type.

---

h/t to my colleague [Stephen Delfick](https://delfick.com)
for pointing this out to me
and saving me from my assumptions.

Thanks to my colleague [Charlie Denton](https:/meshy.co.uk)
for proof-reading this post
and suggesting some clarifying improvements.
