---
title: "How to solve `SSLCertVerificationError`"
date: 2025-02-20
tags:
-   python
---

I follow [Glyph's advice](https://blog.glyph.im/2023/08/get-your-mac-python-from-python-dot-org.html)
and get my Pythons from <https://python.org>.
This works great
and I almost never have problems,
except from occasionally not being able to send HTTP requests
because I get a certificate error:

```python
ssl.SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1006)
```

Thankfully, this is easily remedied.
For Python 3.11,
I resolve the issue with:

```shell
PIP_REQUIRE_VIRTUALENV=0 /Applications/Python\ 3.11/Install\ Certificates.command
```

The `PIP_REQUIRE_VIRTUALENV` is required
because the script uses the Python executable
outwith a virtual environment
and tries to install the `certifi` package
to provide the necessary certificates.
This means that I end up with `certifi` installed in user packages:

```console
$ /Library/Frameworks/Python.framework/Versions/3.11/bin/python3.11 -m pip list
Package    Version
---------- ---------
certifi    2025.1.31
pip        24.0
setuptools 65.5.0
```

Although installing project dependencies
without a virtual environment
is frowned upon
(hence `pip` requiring a special environment variable to allow it),
I think this is fine.

---

I'm not entirely sure why I get that error in the first place.
I'm not entirely sure whether I get it when I first install Python
or whether it starts after an upgrade
with [`MOPUp`](https://pypi.org/project/MOPUp/).
I certainly don't get it all of the time,
so there's *something* funny going on somewhere
🤷🏻
