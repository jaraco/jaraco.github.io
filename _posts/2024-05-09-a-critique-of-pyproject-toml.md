---
layout: post
title: A Critique of pyproject.toml
---

I've recently made the transition from `setup.cfg` to `pyproject.toml` across the [suite of projects](https://jaraco.com/projects) I maintain. It was mostly automatable, thanks in large part to the [ini2toml](https://pypi.org/project/ini2toml) project.

After making the transition, I have a fuller understanding of the benefits and detriments of the format, as first proposed in [PEP 621](https://peps.python.org/pep-0621/), though even before the transition, I'd already identified several concerns with a declarative format in a single file.

This article outlines the concerns and challenges present with the design.

## Bias toward static definition

The PEP specifically advocates statically-defined metadata as a primary motivator:

> Encourage users to specify core metadata statically for speed, ease of specification, unambiguity, and deterministic consumption by build back-ends

This motivation is likely based in an assumption that a project's metadata is inherently stable and statically declared and that a developer has sufficient bandwidth to author and maintain the metadata. And while true for managing a handful of projects, these assumptions start to fall apart when operating at larger scale.

Consider the [suite of projects](https://jaraco.com/projects) I maintain. The burden of maintaining these projects and their metadata becomes a real impediment. The [skeleton](https://blog.jaraco.com/skeleton/) project attempts to address this need by storing the static metadata in a separate repo and composing (merging) that repo with the essential code of each project.

The need for tools like Cookie Cutter or the skeleton demonstrates that the concerns present in the metadata go beyond those of a statically-defined standalone project.

In contrast, enterprise organizations avoid most of this statically-defined metadata, demonstrating an approach that limits friction and scales better.

## Resistance to composition

The fact that `pyproject.toml` is a single file in the root of the repository and stored in version control means that it can't be easily composed to address multiple concerns.

For example, PEP 517 describes how build backends are specified, PEP 621 describes how metadata is indicated, and other projects use `pyproject.toml` for other purposes such as [affecting test behavior](https://github.com/jaraco/pytest-enabler/blob/main/README.rst).

Some projects have even gone so far as to [require configuration](https://github.com/astral-sh/ruff/issues/10299#issuecomment-2004139246) of separate concerns (linting, metadata) to be in a single file or to be specified redundantly in multiple files.

## Ambiguity of order

Since the only method of composition is to have a single, monolithic file, it also means that the order of contents is ambiguous. A file with `[tool.setuptools_scm]` followed by `[tool.pytest-enabler.black]` is functionally identical to one with those sections reversed. The same is true for the fields within a section. As a result, there is ample opportunity for these files to have subtle and inconsequential variance that lead to excessive merge conflicts when using line-based version control operations and lead to arbitrary ordering based on the whims of a particular developer.

## Unbounded scope

And while the arbitrary ordering of metadata is a small concern, as more and more tools support and even recommend use of `pyproject.toml` for configuration, this one file becomes increasingly complex and buries essential signals.

Consider, for example, the [towncrier project config](https://github.com/twisted/towncrier/blob/272993f1f4323db8f96ba67926781d753f207ba7/pyproject.toml). In addition to specifying its `build-system` and `project*` attributes, it also declares configuration for `hatch`, `towncrier`, `black`, `isort`, `ruff`, `mypy`, and `coverage`. It's difficult to see at a high level what tools are used by the project. If this project used `tox` instead of `nox`, the `tox` config might well have also been folded into the `pyproject.toml`, making it difficult to determine what test orchestrator is available.

This unbounded scope increases the complexity of this file and compounds problems like the ambiguity of order.

If the Python metadata supported a model like Unix's `.d`, it would at least allow for better composition, separation of concerns, and reduced ambiguity.

## Overlap with other metadata specs

While PEP 621 describes how to declare the metadata, it's still the responsibility of a backend to read that metadata and then publish it following other specifications. It feels a little bit like [xkcd 927](https://xkcd.com/927/) (Standards).

## Downstream tools overly reliant on its presence

Due to its standardization and proliferation, tools are beginning to *expect* metadata to be present in a `pyproject.toml`, even when such metadata may be dynamic or pyproject.toml may not be desirable at all. There already exist standards (i.e. [PEP 517](https://peps.python.org/pep-0621/)) describing how for a tool to load metadata for a project (built or unbuilt).

As a result, these downstream builders and tools are expecting to be able to load metadata from pyproject.toml even when it's not a relaible source of it and the only reliable way to get it is from the backend-generated metadata.

## Inelegant syntax

At the risk of nitpicking, the toml syntax is inelegnt, especially when compared to something like `setup.cfg`. Consider the [tempora migration](https://github.com/jaraco/tempora/commit/98b0942af3440c751161fd0c870e475c66379f7f) as an example.

Each an every field now requires quotes around the value and fields with multiple values (like Classifiers) require commas separating (or trailing commas for nicer diffs).

The config went from 999 characters in setup.cfg to 1095 characters in pyproject.toml. And while the bytes are cheap, the visual and mechanical aesthetic suffer. Moreover, syntax allows for even more ambiguity now that quoting is a concern. Either of the following dependencies is valid and functionally identical:

```
	"importlib_resources; python_version < '3.12'",
```

```
	'importlib_resources; python_version < "3.12"',
```

Historically, I'd used the more common double-quotes for the specifier (i.e. `python_version < "3.12"`), and `ini2toml` will retain that value and wrap it in single quotes, but I've seen other users switch to using `python_version < '3.12'` in order to maintain consistent double quotes around each of the dependencies.

I miss the `setup.cfg` syntax for its simplicity and elegance, where the developer didn't have to "hand encode" the configuration.

## Overly aggressive normalization

For the `name` field in particular, the spec states:

> Tools SHOULD normalize this name, as specified by [PEP 503](https://peps.python.org/pep-0503/), as soon as it is read for internal consistency.

This recommendation implies that a backend like Setuptools or Flit should normalize the name immediately after reading it from the `pyproject.toml` such that the project's preferred name would never be exposed. In other words, the project named "CherryPy" would have its name stored as "cherrypy" in the metadata, contradicting the stated and defended stance that "CherryPy" is a valid display name for a project. And especially considering that some project names are meaningfully mangled when normalized (e.g. `zope.interface` becomes `zope-interface`), this recommendation represents scope creep for the name normalization.

In practice, backends are not enforcing this recommendation, so perhaps it should just be removed.

## File is largely unnecessary

The existence of a static config runs against the motivations such as those in [this vision](../vision-for-scalable-OSS-development) and demonstrated by [coherent.build](https://pypi.org/project/coherent.build) where there is literally zero static config in the source repository.
