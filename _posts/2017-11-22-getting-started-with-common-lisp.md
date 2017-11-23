---
layout: post
title: "Getting Started with Common Lisp"
description: "A collection of information to getting started with Common Lisp"
category: Programming
tags: [Common-Lisp, Getting-Started]
mathjax:
lang:
trans:
---
{% include JB/setup %}

* toc
{:toc}

## Environment Setup

Start by installing these applications (or some equivalent):

* [SBCL](http://www.sbcl.org/): one of the most popular CL compilers. It can be
    installed by the package manager (like `apt-get install sbcl`).

* Emacs/[Spacemacs](http://spacemacs.org/): a decent editor. For Vim users like
    me, I suggest Spacemacs (check
    [my configuration](https://github.com/boechat107/boringfiles)).
    * [Slimv](https://github.com/kovisoft/slimv) is also a very good
    alternative for Vim users.

* [Quicklisp](https://www.quicklisp.org/beta/): the most popular library
    manager.

{% highlight bash %}
curl -o /tmp/ql.lisp http://beta.quicklisp.org/quicklisp.lisp
## Installs quicklisp at ~/.quicklisp.
sbcl --no-sysinit --no-userinit --load /tmp/ql.lisp \
     --eval '(quicklisp-quickstart:install :path ".quicklisp")' \
     --quit
{% endhighlight %}

{% highlight cl %}
;; Call "sbcl" on terminal.
* (load "~/.quicklisp/setup.lisp")
;; Load quicklisp automatically for every CL session.
* (ql:add-to-init-file)
{% endhighlight %}

* [ASDF](https://common-lisp.net/project/asdf/asdf.html): "a system definition
    facility for Common Lisp programs and libraries". This will be installed
    automatically by Quicklisp just by following the other steps in the next
    sections. However, we need to
    [set it up](http://lisp-lang.org/learn/writing-libraries).

{% highlight cl %}
;; Content of the file ~/.config/common-lisp/source-registry.conf
(:source-registry
  ;; Replace "code" by your preferred directory.
  (:tree (:home "code"))
  :inherit-configuration)
{% endhighlight %}

## New Project

Taken from [Baggers](https://www.youtube.com/watch?v=SPgjgybGb5o):

1. In a terminal, call **sbcl** inside the parent directory of your new project
2. Load **quickproject**: `(ql:quickload :quickproject)`
3. Create the project structure: `(quickproject:make-project "some-project-name")`
4. Check the new directory *some-project-name*

The project structure (or the "system") is defined in the file
**some-project-name.asd**. Follow
[this](http://lisp-lang.org/learn/writing-libraries) reference to understand
better the file.

### Version Management

As far as I know, Quicklisp doesn't understand the dependencies' versions
defined in the *asd* file. However, I like to put their versions down anyway,
like this:

{% highlight cl %}
  :depends-on ((:version "drakma" "2.0.4")
               (:version "jonathan" "0.1")
               (:version "cl-arrows" "0.0.1")
               (:version "ironclad" "0.34"))
{% endhighlight %}

## Conclusion

After some years coding in Clojure, I got really used to the facilities provided
by [Leiningen](https://leiningen.org/) and Maven; for CL, I was surprised that I
couldn't easily find a way to start some serious, well organized project. In
this post I try to summarize all the information I gathered from many different
sources just to start a new simple project.

If you have some better suggestion, please send it to me by email. I would be
really happy to learn something more.

## References

* [http://lisp-lang.org/learn/getting-started/](http://lisp-lang.org/learn/getting-started/)
* [https://www.quicklisp.org/beta/](https://www.quicklisp.org/beta/)
* [https://lispmethods.com/development-environment.html](https://lispmethods.com/development-environment.html)
* [https://www.youtube.com/watch?v=SPgjgybGb5o](https://www.youtube.com/watch?v=SPgjgybGb5o)
