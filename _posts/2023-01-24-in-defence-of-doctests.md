---
layout: post
title: In defence of doctests
---

Many of the projects I maintain in open source make heavy use of doctests.

Therefore, all of lines you see in the docstrings that begin with `>>>` are executed and will fail if the output is not a match for what's written.

I acknowledge that this form of test is discouraged at Google and is suboptimal for many cases, but in this case, it's preferable to rely on the same tests that guarantee the behavior upstream than to try to maintain a fork of the tests in a different form (unittests), getting those correct, and keeping them in sync.

One really nice feature about doctests is that the functionality is consolidated. You can readily see that all of the lines added to the implementation have corresponding tests. By relying on doctests, it also means that the documentation (at least the code portions) is proven correct.

You can see other projects I maintain where I strike the balance between doctests and unit tests (google3/third_party/py/jaraco/functools/) and others where there are nearly no doctests (google3/third_party/py/cherrypy/).
