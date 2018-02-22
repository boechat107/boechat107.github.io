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
    """Makes a POST request to the given "url" using "data"
    as a payload. If some 4xx or 5xx response is gotten,
    the response body is returned as a string; otherwise,
    "None" is returned.
    * url: string
    * data: dictionary
    """
    ...

## Version 2
## This doesn't mean that documentation is not important,
## I'm just stressing the difference.
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

As an example, suppose your application stores some kind of request into a
database and each one has its own state. Usually we want to guarantee that
nobody accidentally insert invalid values and we end up with a code similar to
this one:

```python
def dbexec(data0, data1):
    "A function from a database library."
    pass

STATE_0 = 'posted'
STATE_1 = 'processing'
STATE_2 = 'finished'

def update_state(request, state):
    if state not in set([STATE_0, STATE_1, STATE_2]):
        raise(Exception("Invalid request state"))
    dbexec(request['id'], state)
```

If a new state constant is added, the `set` in `update_state` also needs to be
updated. We can improve this code and make it safer by using an **Enum** class
to enumerate the valid request states:

```python
from enum import Enum

def dbexec(data0, data1):
    "A function from a database library."
    pass

class ReqState(Enum):
    STATE_0 = 'posted'
    STATE_1 = 'processing'
    STATE_2 = 'finished'

def update_state(request, state):
    if not isinstance(state, ReqState):
        raise(Exception("Invalid request state"))
    dbexec(request['id'], state.value)
```

Although it's better than before, we still can only rely on runtime checking.
Mypy give us the possibility of checking the code before running it. Using type
annotations, the code would like like this:

```python
from enum import Enum

def dbexec(data0, data1):
    "A function from a database library."
    pass

class ReqState(Enum):
    STATE_0 = 'posted'
    STATE_1 = 'processing'
    STATE_2 = 'finished'


def update_state(request: dict, state: ReqState):
    ## Running Mypy before starting the application makes
    ## this checking unnecessary.
    # if not isinstance(state, ReqState):
    #        raise(Exception("Invalid request state"))
    dbexec(request['id'], state.value)
```

Now the *state* validation is done before runtime, making the previous
`isinstance` statement unnecessary and the code base safer against many common
stupid bugs. Of course, if the *state* is expected to come from some IO
operation, the runtime checking can't be avoided.

Another interesting detail is the type definition of the argument `request`. We
can be as much specific as we want about a type.
It says that a `request` must be a dictionary, but it could also say that it
is a dictionary like `Dict[str, int]` (string keys, integer values) or even
define a very specific structure using a `dict` like class. Or we could even
don't specify any type at all and still have `state` being statically checked.

## Usage

### Annotation syntax

[Mypy syntax cheat sheet](http://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html)
is a really good reference, all the most used stuff in a single page.

### Running Mypy

After
[installing Mypy](http://mypy.readthedocs.io/en/latest/getting_started.html),
you just need to run it in your project directory
([flags explanation](http://mypy.readthedocs.io/en/latest/command_line.html#additional-command-line-flags)):

```bash
mypy --ignore-missing-imports project-dir/
```

Mypy can also be used as a linter in Vim using
[Syntastic](https://github.com/vim-syntastic/syntastic).

### Add it to your CI

This is one of the most useful ways to use Mypy: a test stage in your CI
pipeline. I use to add it as a step just before the tests execution; if the type
checking fails, I don't even bother to run the tests and the pipeline is marked
as failed.


## Conclusion

* why don't you go with a statically typed language?
    * Python and its vast number of libraries, specially for machine learning.


## References

* https://docs.python.org/3.5/library/typing.html
* https://www.python.org/dev/peps/pep-0484/
* http://mypy-lang.org
* http://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html
