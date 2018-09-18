---
layout: post
title: Call for a Native Service Environment Orchestrator
tags: python, systems
---

Consider a (presumably common) scenario where you have an application (MyApp) that relies on one or more other Python services (Svc1, Svc2). You want to run your application against instances of those services.

You need a way to orchestrate the setup of those services prior to starting your app (for testing, development, etc).

You could do it manually, maybe write a README that describes for a developer how to set up the virtualenvs, making assumptions and leaving it to the developer to adapt the approach to their peculiar environment. You might say how Svc1 needs Python2.7 and Svc2 needs Python3.5 or later and the app should be run/tested under both Python 2.7 and 3.6.

You might write a makefile or pavement file to automate this process, but you'll need to do this for each and every application you develop, adding peculiarities to each and rarely inheriting advances from one to the next.

Some tools in the Python ecosystem help with this need, but they also fall short.

- pytest provides a nice "fixtures" mechanism for describing the dependencies at a granular level (per test even) and resolving those resources on demand, but it doesn't help with setting up the environments necessary for those resources, and it doesn't provide a way to set up those services/fixtures [outside of a test run](https://github.com/pytest-dev/pytest/issues/3817).
- virtualenv provides a nice way of building environments with varied Python versions, but it doesn't help with describing which environments, where they should be created, or what Python version is relevant for a particular environment, and doesn't orchestrate anything.
- pipenv provides a nice interface for describing one environment, creating that environment, and tracking dependencies, but doesn't help if you have more than one environment you want to declare/manage. It doesn't manage dependencies or relationships across environments nor service execution.
- tox wraps virtualenv and venv to provide a declarative approach for managing multiple environments, declaring the python version, dependencies, and other environmental concerns. It does, however, come with other implicit assumptions (typically that it's not meant as an environment build tool, but is meant as a test orchestrator for a single project), so is somewhat clumsy for an environment builder (by default wants to `install .`). It provides no information about relationships across environments.

I have managed to cobble together something of a solution using pytest fixtures which themselves [build on tox environments](https://github.com/jaraco/jaraco.services/blob/2c19c7501ff523feb69b209a5acdd2707608616f/jaraco/services/envs.py#L74-L86), and then start up and tear down the services in dependency order. An application wishing to use this approach needs only to define the local arrangement of services in a tox.ini file, something like:

```
[tox]
skipsdist=True

[testenv]
setenv =
    PIP_INDEX_URL=https://private.server

[testenv:Svc1]
basepython = python2.7
deps = service1.server
skip_install = True
usedevelop = False

[testenv:Svc2]
basepython = python3  # 3.5 or later please
deps = service2[ssl]
skip_install = True
usedevelop = False
```

This approach works pretty well, and is imminently flexible, running the various Python services natively on the host in a cross-platform friendly way.

Much better would be a system that would:

- Extract or replicate the benefits of the pytest fixtures for allowing a piece of Python code to demand a service (or services), functionality to be is provided by third-party libraries by way of entry points.
- Extract or replicate the environment building and declarative configuration of tox.
- Build on the successes of pipenv to build and manage these environments and make them repeatable.
- Not assume Python as the expected environment (support environments of Node, Ruby, etc. and even hybrid environments).

## What about Docker Compose?

Another approach that seeks to address the above use-case is Docker Compose. And in fact, Docker Compose addresses many of these concerns, providing a declarative way to describe the various environments and immense flexibility around building the applications. The big drawback to Docker Compose is it depends on Docker, so each service is running in an isolated environment, often on a different operating system than the developer (usually Linux). Additionally, there's little fine-grained access for something like a unit test to declare a dependency on such a service and have it spun up on demand (or omitted if unused).
