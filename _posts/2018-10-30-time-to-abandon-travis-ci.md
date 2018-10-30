---
layout: post
title: Time to Abandon Travis-CI
tags: python, systems
---

Today, I once again ran up against a [failed build](https://travis-ci.org/yougov/mongo-connector/builds/448124483) due to [incomplete support for Ubuntu Xenial](https://github.com/travis-ci/travis-ci/issues/6469), an operating that's well over two years old now.

A continuous integration system needs to have progressive platform support. Applications and libraries need to test with late releases of operating systems and language releases, preferably with pre-releases, so that a project can validate with those platforms prior to their release.

With Travis, it seems many projects are stuck with an operating system from over four years ago and a Python version that's well over a year old and does not include the latest stable release.

This makes Travis-CI an unacceptable solution for CI.

I've been patient and willing to help, but the frozen answer is that they're not really working on it.

It's time to find something better.
