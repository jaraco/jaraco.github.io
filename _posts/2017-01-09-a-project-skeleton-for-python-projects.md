---
layout: post
title: A Project Skeleton for Python Projects
---

As a software engineer, I'm always looking for ways to optimize the tedious tasks and repetition out of my workflow, and one of these repetitive tasks is building and maintaining Python packages - you know, the repositories that get uploaded into devpi or PyPI as libraries or applications.

Early on, I would simply copy/paste files from one repo to another. I'd grab a setup.py from a project I recently worked on. Maybe I would grab a tox.ini from a post someone wrote or a setup.cfg from a wiki on packaging.

At one point, I found even this copy/paste to be tedious and error prone. I would make an update in one of the projects, but that update wouldn't have any effect on the other projects derived from the same idea.

I set out to write a routine to mechanize the generation of this scaffolding. I implemented it in [jaraco.develop](https://pypi.org/project/jaraco.develop), but intended for it eventually to apply to YouGov projects just as well. The routine ([seen here](https://github.com/jaraco/jaraco.develop/blob/3.0/jaraco/develop/namespace.py#L50-L117)) would generate many of the common assets, injecting the project name and even adjusting for tabs or spaces at the whim of the invocation.

This technique has been [repeated by others](https://pypi.org/project/cookiecutter/) (and probably better) as well. But I found that as I was able to mechanically generate the scaffolding, and new projects would always get the latest and greatest packaging concepts, my growing [portfolio of projects](https://pypi.org/user/jaraco/) would have an ever longer tail of packages missing those improvements.

So I started down the path of keeping the generated skeleton in a separate branch of the repository, and every time the scaffolding needed a refresh, I would re-generate the skeleton in that branch, commit the changes, then merge them with the master branch of the project (keeping separate the changes from the scaffolding with the tweaks unique to the project).

## An SCM managed approach

And this technique worked well enough, but I quickly realized that I could optimize this process further and eliminate the generation step entirely by maintaining this scaffolding branch in a separate repo. [jaraco/skeleton](https://github.com/jaraco/skeleton) was born and now forms the foundation of nearly all of my open-source projects. In the history for that project, you can see directly the evolution of the scaffolding. It's trivially easy for another user or team to fork the repo and adapt it to their liking.

And that's what I've done at YouGov where we have a fork of jaraco/skeleton customized to suit the unique environment of that team. Changes to the upstream project can be readily incorporated through a SCM _merge_ operation.

## Usage (new)

To use skeleton for a new project, simply pull the skeleton into a new project:

    $ git init my-new-project
    $ cd my-new-project
    $ git pull gh://jaraco/skeleton

For convenience, track the skeleton in its own branch:

    $ git branch skeleton

Now customize the project to suit your individual project needs.

## Usage (existing)

If you have an existing project, you can still incorporate the skeleton by merging it into the codebase.

    $ git checkout --orphan skeleton
    $ git rm --cached -r .
    $ git pull gh://jaraco/skeleton
    $ git checkout master
    $ git merge skeleton

Then, resolve any merge conflict.

## Updating

Now, whenever a change is needed or desired for the general technique for packaging, it can be made in the skeleton project and then merged into the skeleton branch of each of the derived projects as needed. As a result, features and best practices for packaging are centrally maintained and readily trickle into a whole suite of packages. This technique lowers the amount of tedious work necessary to create or maintain a project, and coupled with other techniques like continuous integration and deployment, lowers the cost of creating and maintaining refined Python projects to just a few, familiar `git` operations.

No longer is the maintenance of packaging scaffolding a significant impediment to the creation or maintenance of Python projects, allowing them to be created on their own merits.
