---
layout: post
title: Efficient use of CI resources
---

GitHub Actions, probably the most popular continuous integration (CI) system for open source projects provides a facility for running a series of jobs across a matrix of factors such as operating system, language version, and more. This mechanism makes it easy for the user to quickly define the factors representing supported systems. For example:

```yaml
      matrix:
        python:
        - "3.11"
        - "3.12"
        platform:
        - ubuntu-latest
        - macos-latest
```

The code above will cause the product of each item in the matrix to generate the jobs:

- Python 3.11, Ubuntu
- Python 3.11, macOS
- Python 3.12, Ubuntu
- Python 3.12, macOS

Extending this idea is easy - to add support for another Python version, simply add that version number to the list and it will be tested across all platforms.

As convenient as this feature is, it also facilitates rapid consumption of resources. A more mature project might have other factors, such as python implementation or have debug and release builds, and probably supports more platforms and Python versions. One could imagine creating this matrix:

```yaml
      matrix:
        python:
        - "3.8"
        - "3.9"
        - "3.10"
        - "3.11"
        - "3.12"
        - "3.13"
        platform:
        - ubuntu-latest
        - macos-latest
        - windows-latest
        - ubuntu-older-lts
        implementation:
        - cpython
        - pypy
        build:
        - debug
        - release
```

To cover every combination of these 14 factors, GitHub Actions will happily generate a matrix of 96 jobs, 50% of which are expensive Windows and macOS machines, and this approach doesn't even cover other platforms like RedHat or Solaris or BSD. Each push or commit or PR will perform lots of redundant work across those 96 jobs, and while each combination does produce a slightly different environmental perspective, there may be some opportunities for optimization. Also note that some combinations are invalid (PyPy is not available but on a few Python versions), so those would need to be excluded.

In order to be better stewards of the resources that GitHub generously provides, the [skeleton](https://github.com/jaraco/skeleton) project and its adopters downstream have realized some efficiencies can be made by making a few assumptions about the matrix. In particular, it's extremely unlikely that a failure will be associated with exactly one job in the matrix. More likely, any given failure will be caught by a given factor. That is, if a failure is due to a Windows-specific concern, it will be caught by any and all jobs running on Windows. Or if a failure is due to an older Python version, it likely will be caught by all Python versions across all platforms and implementations.

Although rare, there are some cases where these factors intersect and a failure might only be detected on an old Python on a particular platform, but these kinds of concerns can be detected by running both the newest and oldest Pythons against each platform. It becomes vanishingly unlikely that an emergent defect will exist only on an intermediate Python within a specific platform.

Based on these assumptions, skeleton uses the following pattern for its matrix:

- Test the latest and earliest supported Pythons across each platform.
- Test all supported Pythons on Ubuntu.
- Test against a late or latest PyPy.

```yaml
      matrix:
        python:
        - "3.8"
        - "3.12"
        platform:
        - ubuntu-latest
        - macos-latest
        - windows-latest
        include:
        - python: "3.9"
          platform: ubuntu-latest
        - python: "3.10"
          platform: ubuntu-latest
        - python: "3.11"
          platform: ubuntu-latest
        - python: pypy3.10
          platform: ubuntu-latest
```


And for half of the year, between when the newest version of Python is released in beta until it's released and stable across most projects, also include that version across all platforms.

This approach produces a much more modest set of 10 or 13 jobs that cover 99.5% or better of the supported combinations (anecdotally, there have been far less than 1 in 200 false positives) while limiting the amount of resources consumed (and in many cases wasted).
