---
layout: post
title: Vision for Scalable OSS Development
---

Open source software (OSS) development is vibrant and exciting, with enterprise-grade continuous integration, source code management, and even some AI features like Copilot. Leveraging techniques like an [SCM-managed skeleton](https://blog.jaraco.com/a-project-skeleton-for-python-projects/) and [auto-enabling plugins](https://github.com/jaraco/pytest-enabler) has enabled some scalability, but leaves much to be desired. In particular:

- Developers find themselves toiling away on ancillary concerns - project packaging, distribution, linting, etc..
- These concerns are managed for each and every project, multiplying the effort required for maintenance.
- Each repo is [littered with files](https://github.com/jaraco/skeleton) defining the configuration for these ancillary concerns, making the source code a secondary concern and even incentivizing moving it to its own `src` folder.
- Cross-project changes in these ancillary concerns require some form of static synchronization to each and every project. This process is only scalable when most projects are afforded their own dedicated maintainer or maintenance team.
- This inversion of priorities gives ancillary concerns prominence while hiding the primary value of the project under repetitive names. This repetition is apparent in the URL for the [primary source for jaraco.packaging](https://github.com/jaraco/jaraco.packaging/tree/main/jaraco/packaging), where "jaraco" appears three times and "packaging" appears twice, not counting the references in static metadata.
- This repetitive structure limits the ways that the source can be composed. That is, because the repo root is oriented toward the project and not the source, it adds depths to the tree that make it impossible, for example, to put the source code of two projects side by side. As a result, all source must go through a build/distribution process to flatten the source artifacts into something usable and importable.

Having recently spent several years on development at Google, including a stint working on Google Source, I've come to appreciate some of the key characteristics of software development at perhaps the largest scale humanity has every seen:

- The monorepo concept means that every piece of software across the enterprise is developed in the same history. Coordination and synchronization happens naturally as part of committing code.
- The source code is given primacy and organized around each project's essential location. For example, [tempora](https://github.com/jaraco/tempora) project will appear as `/third_party/py/tempora` with `__init__.py`, `schedule.py`, etc. directly in that folder. As a result, the code appears the way it will be installed and for pure-python projects.
- The bulk of the cross-project concerns are managed outside of the project's source and typically managed in infrastructure. For example, a project that relies on pytest for tests will declare "pytest-tests" in the BUILD instructions along with dependencies, but details about how to build it (locally or for a hermetic build) are implied by infrastructure and tooling instead of in `tox` and `.github/workflows/*`.
- All change logging happens through commit messages. There's no need to manage or curate a "change log" beyond the commits to the source.
- Builds are largely repeatable, since all tooling and source are determined by the state of the repo at the time of the build.
- All artifacts are built from source, so there's no need for artifact repositories and the maintenance and coordination friction that entails.

This approach to organizing code means that developers can focus primarily on the value proposition of the code and less on managing the ancillary details across each and every project. Best practices can be developed separately, in the tooling and infrastructure. And when those advances are made, they are applied instantly across all projects (the "build at HEAD" philosophy).

This approach drastically reduces the need for boilerplate and repetitive configuration. It works toward a vision of "zero config" projects, where the presence of the code is sufficient. Imagine a ecosystem where authoring and publishing a hello world project is as simple as:

```shell
 @ mkdir -p depot/py/world
 @ cd depot/py/world
 @ git init .
 @ cat > hello.py
__name__ == '__main__' and print("hello world")
 @ git add .
 @ git commit -m "Created the world package with hello module"
 @ git push  # or gh pr create ...
```

A sufficiently-knowledgable system could from that repo, infer the following metadata:

Name: world  # inferred from parent directory name or git metadata
Description: A hello world library  # drawn from GitHub description (if present)
Version: 0.0.1  # based on tags in the repo
Author-email: "Jason R. Coombs" <jaraco@jaraco.com>  # inferred from commit history
Classifier: Development Status :: 4 - Beta  # based on version
Classifier: Programming Language :: Python :: 3  # inferred from source
Classifier: License :: OSI Approved :: MIT License  # default defined in tooling
Python-Requires: >= 3.8  # defaults to Python lifecycle supported versions
Project-Url: Homepage, https://github.com/jaraco/world  # drawn from git origin

The system could additionally provide tooling to build/test/publish/publish docs/etc.. A bit of sophistication in GitHub could enable continuous integration and testing based on best practices, inferred capabilities, and selective project-specific overrides, all without requiring statically-defined boilerplate in each project.

For cases where metadata is legitimately needed, that metadata should be stored _away_ from the primary source. I suggest it should be stored in a path that's indicative of being non-source, something like: `(meta)/`. Such a path would never be importable and would be clearly not source. It avoids perpetuating the abuse of the platform-sensitive "dot-prefixed" folders. It's language agnostic, so could readily apply to projects outside the Python ecosystem.

Such a design would allow distribution packagers and downstream enterprise consumers to readily adopt the primary source, even using git features like [submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) to compose projects into a source of truth for an application:

```shell
 @ git init my-amazing-app
 @ cd my-amazing-app
 my-amazing-app main @ git submodule add -q gh://jaraco/tempora
 my-amazing-app main @ git submodule add -q gh://cherrypy/cherrypy
 my-amazing-app main @ git commit -a -m "Added tempora and setuptools"
```

Incorporating such projects into an enterprise project would be as simple as copying over the code to their essential locations and potentially re-writing the metadata (or possibly honoring it directly). Organizations wishing to have a "monorepo" experience could readily do so by composing their source alongside submodules of open source packages.

There are still some tricky edge cases to work out, especially for namespace packages (where the Python package `jaraco.packaging` should appear under `jaraco` alongside other packages of the same namespace, so to make those projects similarly composable, they will need to have their code also prominently at the top level and be composed within a `jaraco` folder).

Over the coming weeks, I plan to develop a prototype of this system at github.com/coherent-oss, tentatively dubbed the "Coherent Software Development System". [Follow along and discuss here](https://github.com/coherent-oss/roadmap).
