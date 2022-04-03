---
layout: post
title: In Python, use docstrings or comments?
date: '2022-01-01T15:26:00.000-05:00'
---

Twenty years ago, [PEP 257](https://www.python.org/dev/peps/pep-0257/) introduced docstrings. Yet, it's [still not obvious](https://bugs.python.org/msg409159) that docstrings are nice and preferable to comments.

# Advantages

There are several reasons to prefer docstrings to comments in Python code in general.

## Canonical description

The primary advantage of a docstring is that Python recognizes the docstring as the primary means of providing a description of the functionality. In fact, the [docs for functions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions) state:

> it’s good practice to include docstrings in code that you write, so make a habit of it.

The docstring convention provides a well-defined place to describe the behavior of the function or method.

## Recommended convention

The [top answer](https://stackoverflow.com/a/2357251/70170) to a related question in Stackoverflow shows a strong consensus toward preferring docstrings to comments.

## Multiline syntax

Docstrings inherently support a multi-line syntax. In particular, docstrings are [recommended to use triple quotes](https://www.python.org/dev/peps/pep-0257/#id9), even when the docstring is a single line, in order to facilitate easy editing to include multiple lines.

In contrast, comments in Python follow the shell-style comments that only apply to a single line. There is no syntax to create a comment in Python that consists of multiple lines, other than to create multiple adjacent comments, which causes the comment characters to be interspersed with the intrinsic message of the comment.

As a result, it's much easier for an IDE or sophisticated editor to support descriptions that extend beyond the recommended wrap length ([72 characters](https://www.python.org/dev/peps/pep-0008/#id17)) to readily wrap and reflow text that's in a docstring than what's in a comment. Or for a human agent using a primitive editor, it's much less cumbersome to edit these descriptions without having to ensure that comment characters are present at the beginning of each line.

## Runtime visibility

Unlike comments, which are stripped out of the syntax prior to parsing, docstrings are recognized syntax, parsed into the syntax tree, and attached to the relevant objects, making their descriptions readily accessible to the runtime. This feature allows for tooltips in IDEs and supplies meaningful documentation to users through the built-in [help](https://docs.python.org/3/library/functions.html#help) function.

The [docstring guidance](https://docs.python.org/3/tutorial/controlflow.html#documentation-strings) also provides recommended practices for formatting the docstring for consistency across authors, including advice on expectations around indentation on which tools can rely to consume docstrings.

## Clarity of locality

Comments are frequently placed before or at the end of the lines they seek to document, whereas docstrings necessarily must appear after/within the objects they document. As a result, if a developer attempts to document a function with comments rather than docstrings, convention would dictate that the documenting comment for a function would appear before the function definition.

## Separate commentary

Because docstrings are not comments, they provide a clear separation of purpose, allowing for subsequent comments about the implementation to appear unambiguously.

Consider this ambiguous commentary:

```python
def do_something():
    # Do something
    # Run just once
    if not was_run_before():  # noqa: F821
        ...
    ...
```

In this contrived example, the second line of the comment indicates that something should be "Run just once", but it's ambiguous whether that message is part of the docstring of the function or documenting the code below it. Moreover, it's not even obvious if the two lines of comments include documentation for the function at all.

By employing docstrings, commentary on the implementation is unambiguously separated from the function description and in many cases will by syntax highlighted differently for maximal distinction:

```python
def do_something():
    """Do something"""
    # Run just once
    if not was_run_before():  # noqa: F821
        ...
    ...
```

It's conceivable that one could extend the convention around writing comments as function descriptions to disambiguate the two (such as by requiring a blank link between the first and second comment), but that approach only further complicates the landscape of appropriate syntaxes.

# Tests as a Special Case

In [bpo-46126](https://bugs.python.org/issue46126), I reported to the CPython project an issue where the presence of docstrings in tests is creating an incentive to prefer comments over docstrings for tests in CPython and beyond.

While there are several advantages to preferring docstrings to comments in Python code generally, it could be argued that _tests_ are a special class of Python code and thus deserve special consideration and maybe docstrings should be avoided.

Due to the advantages above and also for the sake of simplicity, the conventions around tests should diverge as little as possible from non-test Python code. It's valuable to test authors and maintainers to have a canonical place to provide a description of the test, the expectations it creates, issues associated with the expectations, and other context. There should be a compelling reason to eschew the advantages above and adopt a different convention for tests, which would require developers (and their editors) to maintain a separate context for developing tests than developing other Python code.

It's also unclear where the line should be drawn; should this divergent convention apply only to tests themselves or to neighboring/supporting functionality (fixtures, helper functions, etc)?

The primary (sole?) reason given for preferring comments to docstrings in tests is that some test runners under certain circumstances will not render the name of the test (packages, modules, filenames) if a description (docstring) is present, but will instead render the first line of the description.

For example, in [this message](https://bugs.python.org/issue46126#msg408882), unittest can be seen rendering the description:

```
./python.exe -E -We -m test -v test_importlib | grep '... ERROR'
Entry points should only be exposed for the first package ... ERROR
test test_importlib failed
```

Unittest is trying to be helpful here by providing a more meaningful characterization of the intended/missed expectation as provided by the author. The idea here is that if the author took the time to describe the test expectation, that the person running the test would find that information more valuable than the location of the failure.

The problem comes in that someone seeing that error may not care about what expectation was missed but would care more about where that expectation was implemented for a more thorough investigation (reading the rest of the description, reading the code), and because the other output was elided by `grep`, it's unclear where the test is that failed.

However, most test runners do report the location of the test even when a docstring is supplied, and even unittest does report the location of each failed test using the default runner:

```python
# test_something.py
import unittest


class AnythingTester(unittest.TestCase):
    def test_something():
        """Test that something happens (always fails)"""
        raise ValueError('always fails')
```

```
$ python -m unittest discover
E
======================================================================
ERROR: test_something (test_something.AnythingTester)
Test that something happens (always fails)
----------------------------------------------------------------------
TypeError: AnythingTester.test_something() takes 0 positional arguments but 1 was given

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
```

In this case, Unittest clearly shows both the location of the failed test and the expectation that was missed (first line of the docstring). If that docstring was not present, unittest simply omits that extra detail in the output.

Therefore, it is only when running with `-v` and using `grep` to filter output that the location of the test is not readily available when the error is reported.

That narrow behavior hardly seems like a good enough justification to create a new, divergent convention to apply comments in an unusual location (below the function signature where a docstring would be instead of above where a comment usually goes) to avoid an undesirable behavior in a test runner, especially when that behavior is configurable.
