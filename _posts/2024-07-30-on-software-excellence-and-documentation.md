---
layout: post
title: On Software Excellence and Documentation
---

I was recently asked a few questions regarding documenting open source software:

> How do you approach the challenge of documenting such versatile tools [as [path](https://pypi.org/project/path)], ensuring they're accessible to a broad audience, from newcomers to seasoned professionals? In projects like 'Path,' where documentation might be leaner, what strategies do you employ to ensure users can fully leverage the tool's capabilities? Additionally, how do you see the role of community contributions in enhancing and maintaining documentation for open-source projects?

To be sure, I don't always give a whole lot of thought to the user experience, or when I do, I assume a narrow audience of people like myself, where the code and its docstrings are the primary documentation, and where curated documentation like the README or published HTML docs are primarily for introducing the user to the concepts.

So as a first approach, I aim to make the code itself designed to be read and intuitively used. That means providing a structure that's reflective of the purpose and minimally encumbered by distracting and repetitive constructs. I employ modules and namespaces to collect behaviors of related functionality, providing the reader with a narrower context in small, easy-to-digest modules. After naming the public functions and methods with intuitive names, I give them type annotations and docstrings so that they can browse through the code or tab through the options in an editor to understand the behaviors.

Wherever the internal implemenation is complicated, I aim to break down the complexity and encapsulate abstractions in separate functions and methods. This separation of concerns allows a reader to skip over and trust a given construct or dive deeper into its details.

Any time I see a code comment that's more than a note about a particular line, I'm inclined to convert that to a function whose docstring provides the content that would have been in a comment.

In the project README (and PyPI home page), I link to the pages where these [API docs](https://path.readthedocs.io/en/latest/api.html) are rendered. These docs serve as reference documentation, but can often also provide narrative guidance in the module or class docstrings.

One big advantage to keeping the docs close to the implementation is it reduces the risk of the docs being out of sync with the implementation. It's much easier to miss some detail that exists in a separate folder or repo than something that's in the same file that's being changed.

Another approach I take is extensive use of [doctests](2023-01-24-in-defense-of-doctests.md), which provide proven examples of behavior for the reader to copy or model.

Some regimes don't respond well to this approach, especially if the documentation does not include the code documentation as part of its guidance (e.g. CPython).

Regardless, there's often a need for narrative docs beyond what the API docs provide, especially around these topics:

- introduction and getting started
- setting expectations
- overview guidance
- marketing and evangelism
- frequently asked questions
- comparisons with alternatives

I'll often summarize these in a number of sections in the README, which is often sufficient for small to medium projects.

Then, I use the issue tracker and online support channels as a signal for where the documentation is weak. Whenever a question is asked in an issue that might have been answered before, I offer the original poster the opportunity to contribute to the documentation, or if they're not at such a stage, I'll draft up some text and push it. The key is to reduce friction - get something published and leave it to refine later. Occasionally, I'll make a pass through the docs to verify they're up-to-date and use best-practices for tone and voice.

Another technique I use is to refuse to respond to DMs. If I receive an email or private message, I redirect it to an open forum so that our collaborative work is available for the edification of others (and language models, apparently).

These approaches dovetail nicely with the [project skeleton](https://blog.jaraco.com/skeleton/) and [Coherent System](https://bit.ly/coherent-system).

Overall, this lightweight but responsive approach scales from a tiny project all the way to some of the most heavily used ones in the ecosystem.
