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
Python 3, providing the fundamental building blocks for constructing types.
Although these types are simple hints and have
[no semantics defined](https://www.python.org/dev/peps/pep-0484/#abstract) in
the language, third party libraries can make use of them to do static type
checking.

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

As well mentioned in a
[Zulip's blog post](http://blog.zulip.org/2016/10/13/static-types-in-python-oh-mypy/),
readability is one the biggest and immediate benefits of annotating types for
statically type checking.
Even when using dynamic languages, it's a good practice to describe, whenever
possible, the signature of a function
([*How to Design Programs*](https://is.gd/iF59HL) examples), noting the
input and output types. This simple practice can save a lot of time when
debugging a code, since you wouldn't need to trace back a chain of function
calls to determine the type of some data.

However, if the signature of functions can be only described by *comments* or
*docstrings*, we have another possible point of failure: buggy signatures. It is
very common to change the input/output types of functions over its lifetime,
but there is no guarantee that their documentation are consistent if they are
not automatically checked.

For example, let's consider the function argument `s3path` being a string like
`s3://my-bucket/path/to/key`:

```python
from typing import List

def split_bucket_key(s3path: str) -> List[str]:
    return s3path.split('//')[1].split('/', maxsplit=1)
```

This code successfully passes the type checking. But then you decide that
returning a **tuple** makes a better interface and you accidentally forget to
update the type annotation:

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


### Fewer Unit Tests

Given enough type annotations, unit tests composed of runtime type checking
could be just replaced by the general static type checking done by Mypy.
The type inference engine could spot bugs in many different places, specially as
new code is added to the code base.

The examples in the next subsections illustrate this idea.


### Debugging

In addition to readability, describing function signatures helps debugging,
specially if you can trust this information. Instead of worrying about types and
values, we can focus only on values, and this can be a tremendous difference.

In the example below we have two versions of an hypothetical function whose
purpose is to make a POST request. The first version relies entirely on
*docstrings* as its documentation, while the second uses type annotation that
can be checked using Mypy.

```python
import random
from typing import Optional

## Version 1
def post_some_data(url, data):
    """Makes a POST request to the given "url" using "data"
    as a payload and returns the response's body as a string.
    If some error occurs (4xx or 5xx response), "None" is
    returned.
    * url: string
    * data: dictionary
    """
    # Make the real request...
    return random.choice(['something useful', None])

## Version 2
def post_some_data(url: str, data: dict) -> Optional[str]:
    """Returns the response's body for successful requests;
    otherwise, returns "None".
    """
    # Make the real request...
    return random.choice(['something useful', None])
```

Suppose somewhere in our code base we use this function, but we forget that it
not always returns a string:

```python
resp_err = post_some_data('www.blabla.com', {'key': 'value'})
# An accidental mistake.
resp_err.split(' ')
```

Using the first version, we could only notice this bug when a request fails, at
runtime (or we can never get it at all):

```python
Traceback (most recent call last):
  File "pycode.py", line 9, in <module>
    a.split(' ')
AttributeError: 'NoneType' object has no attribute 'split'
```

If we write the same buggy code using the second version and call Mypy, we
instantly get the bug without even executing the code:

```bash
$ mypy --strict-optional pycode.py

pycode.py:8: error: Item "None" of "Optional[str]" has no
attribute "split"
```


### Gradual Typing

This seems to be a kind of
[official term](https://en.wikipedia.org/wiki/Gradual_typing) to represent type
systems in which dynamic and static types are used together. Some parts could be
type checked at *compile time* (or *non-runtime* in Python) and others only at
runtime. Mypy supports this mixed approach; it keeps the freedom to those who
likes the Python dynamic nature to quickly prototype ideas while providing tools
to confirm the correctness (in terms of types) of critical parts of an
application.

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
updated. We can improve this code and make it safer by using an `Enum` class
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

Although it's better than before, we still can only rely on runtime checking and
unit tests.
Mypy give us the possibility of checking the code before running it. Using type
annotations, the code would be like this:

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
`isinstance` statement unnecessary and the code base safer against many common,
accidental bugs. Of course, if the *state* is expected to come from some IO
operation, the runtime checking can't be avoided.

Another interesting detail is the type definition of the argument `request`. We
can be as much specific as we want about a type.
It says that a `request` must be a dictionary, but it could also say that it
shall be a dictionary like `Dict[str, int]` (string keys, integer values) or
even define a very specific structure using a `dict` like class. Or we could
even don't specify any type at all and still have `state` being statically
checked (as it is done with the *return* type).

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
mypy --ignore-missing-imports --strict-optional project-dir/
```

Mypy can also be used as a linter in Vim using
[Syntastic](https://github.com/vim-syntastic/syntastic).

### Add it to your CI

This is one of the most useful ways to use Mypy: a test stage in your CI
pipeline. I use to add it as a step just before the tests execution; if the type
checking fails, I don't even bother to run the tests and the pipeline is marked
as *failed*.


## Conclusion

The inclusion of type annotation syntax and the development of a static type
checker such as Mypy are a great step the Python community has taken towards
safer and clearer code, without giving up the language's dynamic nature.


## References

* [https://docs.python.org/3.5/library/typing.html](https://docs.python.org/3.5/library/typing.html)
* [https://www.python.org/dev/peps/pep-0484/](https://www.python.org/dev/peps/pep-0484/)
* [http://mypy-lang.org](http://mypy-lang.org)
* [http://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html](http://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html)
* [How to Design Programs](http://www.ccs.neu.edu/home/matthias/HtDP2e/index.html)
