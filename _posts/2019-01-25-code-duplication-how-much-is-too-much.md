---
layout: post
title: Code Duplication - How Much Is Too Much?
tags: programming, software-development
---

Coming out of University, having taken Cleanroom Development and Software Construction courses, I had a fairly regimented DRY view on duplication of code - never do it.

Joining YouGov, I was enticed to the idea, "Why not just copy it? You can refactor later." I've tried that concept in a number of different domains and scopes and it's always come back to haunt me.

Today, I encountered a particularly stark [real-world example](https://github.com/jaraco/configparser/blob/0e8135cd5d6a5de5c5c5e6a80dbbc19d5e314a21/src/backports/configparser/__init__.py#L687-L700). Consider this code:

```python
if PY2 and isinstance(filenames, bytes):
    # we allow for a little unholy magic for Python 2 so that
    # people not using unicode_literals can still use the library
    # conveniently
    warnings.warn(
        "You passed a bytestring as `filenames`. This will not work"
        " on Python 3. Use `cp.read_file()` or switch to using Unicode"
        " strings across the board.",
        DeprecationWarning,
        stacklevel=2,
    )
    filenames = [filenames]
elif isinstance(filenames, str):
    filenames = [filenames]
```

In this example, the duplicated code is one line, `filenames = [filenames]`. Seems pretty innocuous, right? Can you anticipate what problem one might encounter with this approach?

The problem is that the behavior is duplicated in two branches of the code, and the branches of code are somewhat asymmetrical. One may want to affect the warning, for example, but not affect the list construction. And in [this pull request](https://github.com/jaraco/configparser/pull/24), that exact situation happened.

Now granted, a good test suite would have had good coverage for all of the different cases, but it didn't, and the project had a broken release. If instead the code had been structured such that the `filenames = [filenames]` was gated just once on the relevant conditions (in exactly one place) and the warning was gated on its relevant conditions, such a mistake and the ensuing outages would have been avoided.

Consider instead:

```python
# we allow for a little unholy magic for Python 2 so that
# people not using unicode_literals can still use the library
# conveniently
py2_bytes = PY2 and isinstance(filenames, bytes)
if py2_bytes:
    warnings.warn(
        "You passed a bytestring as `filenames`. This will not work"
        " on Python 3. Use `cp.read_file()` or switch to using Unicode"
        " strings across the board.",
        DeprecationWarning,
        stacklevel=2,
    )

if isinstance(filenames, str) or py2_bytes:
    filenames = [filenames]
```

This form has a single conditional describing the "filenames to list" concern and a separate one for the warning. Had the original code taken this form, a pull request to affect the warning would be far less likely to affect the "filenames to list" concern. By writing it this way, another shortcoming is more apparent - the warning won't occur when a `bytes` filename is passed in a list.

And in fact, it was my instinct to [unify the logic here](https://github.com/jaraco/configparser/commit/5ce1d1a7cb5781b94f405bfed8c77a4ac2cec8b5#diff-3fd1a3ced031110564f8eb79be10b30eR700), albeit by that time the code had evolved enough that the `bytes` consideration was unconditional on Python version.

The lesson I take from the failure reinforces again the importance of factoring code in such a way as to separate concerns and avoid repetition, even for a single line.

Perhaps even better would be to rely on something like [always_iterable](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.always_iterable) to handle the 'single or iterable' pattern in a proven, unit-tested manner. Of course, one does have to consider the maintenance burden of adding the dependency, which is high especially for packages that have no other dependencies or have broad compatibility requirements. In that case, a copy-paste referencing the canonical implementation is an acceptable tradeoff.

If the duplication of a single line of code can cause this much damage, just imagine how much greater the magnitude of harm when it's a whole function or module or package or company that's forked (particularly without a direct link to its ancestor and constraint on change). Often the manifested harm isn't an error or outage, but a cognitive burden or unbridled ballooning complexity that reduce velocity and productivity.
