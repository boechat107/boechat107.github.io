---
layout: single
title: "My Clojure programming environment"
description: "What I use to code in Clojure using Vim."
category: Programming
tags: [Clojure, Vim]
---


Until the last week, [Slimv](https://bitbucket.org/kovisoft/slimv) was my main tool to
code in Clojure using Vim. Although Slimv is still a very nice plugin, it seems not 
a good idea to keep using a swank server to run a REPL. 
[Fireplace](https://github.com/tpope/vim-fireplace) and 
[nREPL](https://github.com/clojure/tools.nrepl) are the recommended tools to work 
with Clojure and Vim, but to have similar features of Slimv, I needed to add another
tools and plugins. These below are what I found:

* [paredit.vim](https://github.com/vim-scripts/paredit.vim), the same paredit 
functionality of Slimv.
* [rainbow_parentheses](https://github.com/kien/rainbow_parentheses.vim)
* [redl](https://github.com/dgrnbrg/redl) and its dependencies, which gives us a
real REPL.

*Update:*

Recently I needed to use Common Lisp for a task and I found that Slimv works almost
perfectly! So, I just removed the paredit and rainbow plugins, since Slimv has them
as built-in features, and kept Slimv working with Fireplace and redl.

Additional plugins:

* [cljfold](https://github.com/gberenfield/cljfold.vim), for automatic code folding.

