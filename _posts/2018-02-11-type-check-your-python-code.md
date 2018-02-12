---
layout: single
title: "Type Check Your Python Code"
description: "Type Check Your Python Code"
category: Programming
tags: [Python, Type System]
mathjax:
---

I was recently surprised to discover that a module called
[**typing**](https://docs.python.org/3.5/library/typing.html) had been added to
Python 3, providing the fundamental building for constructing types. Although
these types are simple hints and have
[no semantics defined](https://www.python.org/dev/peps/pep-0484/#abstract) in
the language, third party libraries can make use of them to do static type
checking working like a linter.

A project called [**Mypy**](http://mypy-lang.org/about.html), initially developed
by a Dropbox team, is able to statically check Python code using a powerful type
system with very interesting and useful features, like [bidirectional type
inference](https://people.mpi-sws.org/~joshua/bitype.pdf) and
[generics](http://mypy.readthedocs.io/en/latest/generics.html).

If you have some experience with ML languages like Haskell, OCaml or F#, you can
probably already see the great value of such a tool; if you don't, I'm going to
point out a few benefits and reference some resources for a better
understanding.


## Benefits

These are just a few benefits, the ones I observed quickly after starting using
Mypy.

### Readability

As well cited in a
[Zulip's blog post](http://blog.zulip.org/2016/10/13/static-types-in-python-oh-mypy/),
readability is one the biggest and immediate benefits of statically type checking
the code. In many situations, good function and argument names, together with
their types, suffice to make clear their intentions. Of course this could also
be done as a docstring, but if you update the function you need to remember to
check if the docstring is still consistent.

For example, considering a `s3path` being a string like
`s3://my-bucket/path/to/key`:

```python
from typing import List

def split_bucket_key(s3path: str) -> List[str]:
    return s3path.split('//')[1].split('/', maxsplit=1)
```

This code successfully passes the type checking. But then you decide that
returning a **tuple** makes a better interface:

```python
from typing import List

def split_bucket_key(s3path: str) -> List[str]:
    return tuple(
        s3path.split('//')[1].split('/', maxsplit=1)
    )
```

Mypy would correctly complain about your types:

```
pycode.py:4: error: Incompatible return value type
(got "Tuple[str, ...]", expected "List[str]")
```




### Fewer Tests

Given enough type annotations, some simple unit tests including type checking
(which would depend on the runtime data) could be automatically replaced by the
static type checker. The type inference engine could spot bugs in many different
places, specially as new code is added to the code base.

Of course, these tests are not usually the most important, but type checking
your code gives you confidence to focus on testing the most important parts of
the application.


### Debugging

How many times have you needed to trace back a chain of functions or
method calls just to be sure about data type?

It's important to note that Mypy, for instance, doesn't require type annotations
for a whole module or project, mixing static and dynamically type code is
expected ([gradual typing](https://en.wikipedia.org/wiki/Gradual_typing)).

* war between dynamic and static
* one should recognize the good points of both sides
* Python is very popular, specially in areas such machine learning
* "gradual typing" and MyPy seems to be bring some of the good points of
    statically typed languages

## Usage

* some code samples
* cli invocation
* cli options (like ignore imports)

## Part of your CI

* example of using Docker with mypy


## Conclusion

* why don't you go with a statically typed language?
    * Python and its vast number of libraries, specially for machine learning.

## References

* https://docs.python.org/3.5/library/typing.html
* http://mypy-lang.org
* https://www.python.org/dev/peps/pep-0484/
