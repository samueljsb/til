---
title: "Use `NonCallableMock` in Django templates"
date: 2026-09-10
tags:
-   django
-   python
---

Django sometimes does surprising things while it's trying to be helpful.
I've been bitten by this particular situation a few times recently
and I'm always puzzled until I remember what's going on.

## The problem

When I'm unit-testing a view in a Django project
I like to provide mocks for the view's dependencies
so I can focus on testing the view itself.
This often means constructing mock objects in my test
with attributes that will be accessed by my view.
Importantly, I *always* spec my mocks:

```python
from unittest import mock

fake_thing = mock.Mock(
    spec=views.Thing,
    attribute='hello world!',
)
```

I might then have a template:

```jinja
{{ thing.attribute }}
```

And my view code returns my fake "thing" in its context.

The problem is that this example,
as presented here,
does something very unexpected.
If I render this template,
the result looks quite strange:

```html
&lt;Mock name=&#x27;mock().attribute&#x27; id=&#x27;4422586720&#x27;&gt;
```

<!-- markdownlint-disable MD033 -->
<details>
<summary>Example code</summary>
<!-- markdownlint-enable MD033 -->

If you want to try this out yourself,
the following code replicates this situation:

```python
from typing import Protocol
from unittest import mock

import django.template


class Thing(Protocol):
    @property
    def attribute(self) -> str: ...


fake_thing = mock.Mock(spec_set=Thing, attribute='hello world!')

engine = django.template.Engine()
template = engine.from_string('{{ thing.attribute }}')
context = django.template.Context({'thing': fake_thing})

print(template.render(context))
```

</details>

## What's going on?

This output is actually HTML-escaped --
what's been inserted into the template is

```text
<Mock name='mock().attribute' id='4422586720'>
```

That's the `repr` for an attribute of a `Mock` object,
but why‽

Django, very helpfully, calls any callable object before using it in a template.
In this case, my `Mock` instance is callable
and will return a *new* `Mock` instance,
which does not have the `attribute` attribute.

## The solution

Fortunately,
the Python standard library has a kind of mock object that *isn't* callable.
It's called [`NonCallableMock`]!

All I have to do is replace the type of my mock:

```diff
- fake_thing = mock.Mock(
+ fake_thing = mock.NonCallableMock(
      spec=views.Thing,
      attribute='hello world',
  )
```

And everything works as I expect it to!

```html
hello world!
```

[`NonCallableMock`]: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.NonCallableMock
