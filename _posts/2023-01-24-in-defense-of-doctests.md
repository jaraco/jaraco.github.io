---
layout: post
title: In defense of doctests
---

Many of the projects I maintain in open source make heavy use of [doctests](https://docs.python.org/3/library/doctest.html). Supplying doctests affords a luxury and elegance not offered by unit tests, namely:

- canonical and representative examples are described alongside publishable guidance, reducing duplication of work and opportunities for inconsistency.
- the documented behavior is guaranteed to be correct (assuming tests are run).
- the syntax is necessarily standardized.
- the instructions can be readily parsed by machines.
- the examples can be mechanically re-written for consistent style.
- behavior and tests can often be consolidated in a single code block for simple patches and easy review.
- encourages stateless functions with simple inputs and outputs.
- doctests contribute to code coverage and often are sufficient to capture the full range of supported use-cases.
- zero config framework; tests can be exercised simply from the standard library without any additional imports.
- tests can be supplied on non-importable modules (such as scripts).

Therefore, lines in the docstrings that begin with `>>>` are executed and will fail if the output is not a match for what's written. I acknowledge that this form of test is often discouraged and is suboptimal for many cases (see [Soapbox](https://docs.python.org/3/library/doctest.html#soapbox)). In particular, if doctests have to spend a high percentage of the user's attention to facilitate testing, it's probably better migrated to a unit test. With judicious use of fixtures and aggressive migration to unit tests when appropriate, doctests can be quite valuable and intuitive. That's why all [skeleton](https://github.com/jaraco/skeleton)-based projects [run doctests as a matter of course](https://github.com/jaraco/skeleton/blob/f9e01d2197d18b2b21976bae6e5b7f90b683bc4f/pytest.ini#L3).

Where these doctests exist in upstream code, it's preferable to exercise those tests as the upstream project does instead of maintaining a port of those doctests to unit tests.

Some projects (e.g. [jaraco.text](https://github.com/jaraco/jaraco.text), [jaraco.context](https://github.com/jaraco/jaraco.context)) rely entirely on doctests. More sophisticated projects strike the balance between doctests and unit tests ([jaraco.functools](https://github.com/jaraco/jaraco.functools), [pip-run](https://github.com/jaraco/pip-run), [irc](https://github.com/jaraco/irc)) and others have nearly no doctests ([cherrypy](https://github.com/cherrypy/cherrypy)).
