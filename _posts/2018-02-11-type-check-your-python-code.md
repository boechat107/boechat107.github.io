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
method calls just to be sure about a data type?

After a few months annotating types, I notice that the debugging started to
become easier and faster.
The type checking helped me avoid many stupid mistakes with `None` values, for
example; sometimes when I forgot to add a return statement (maybe I was too
*Lispy*), sometimes when I just forgot that a `None` was a possible return value
of a function.

An example:

```python
from typing import Optional

## Version 1
def post_some_data(url, data):
    """Makes a POST request to the given "url" using "data" as a payload. If
    some 4xx or 5xx response is gotten, the response body is returned as a
    string; otherwise, "None" is returned.
    * url: string
    * data: dictionary
    """
    ...

## Version 2
## This doesn't mean that documentation is not important, I'm just stressing the
## difference.
def post_some_data(url: str, data: dict) -> Optional[str]:
    ...
```

While debugging your application for an apparently `None` value problem, which
version do you think would help you to avoid the wrong spot? The first one has a
reasonable documentation, but, at least to me, I still feel better about the
second version, specially if the type checker tells me that the types are OK (of
course, this may depend on how much annotation we have). The `Optional` type
tells me that whoever uses this function should be prepared to handle a string
or `None`. How could I be sure that the first version really returns a string?
What if someone updated the code to return some response object and forgot to
update the docstring?


### Gradual Typing

This seems to be a kind of
[official term](https://en.wikipedia.org/wiki/Gradual_typing) to represent type
systems in which dynamic and static types are used together. Some parts could be
type checked at *compile time* (or *non-runtime* in Python) and others only at
runtime. Mypy supports this mixed approach; it keeps the freedom to those who
likes the Python dynamic nature to quickly prototype ideas while providing tools
to confirm the correctness of critical parts of an application.

* example of db.state of lambdazome; initially it was a simple string, then it
    became a type

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
