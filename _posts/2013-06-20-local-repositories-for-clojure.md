---
layout: post
title: "Local repositories for Clojure"
description: "How to set local repositories for Clojure projects."
category: Programming
tags: [Clojure, Leiningen]
---
{% include JB/setup %}

Local repositories are useful when you have some dependencies that are not in Maven
or Clojars repositories. The following steps are one alternative to suppress this
kind of problem, considering that you are using
[Leiningen](https://github.com/technomancy/leiningen) to manage your project.

1. Create a local repository folder inside your project, like `repo`.
2. Use the leiningen plugin [**localrepo**](https://github.com/kumarshantanu/lein-localrepo)
to install a jar into the local repository.
3. Add the option `:local-repo "repo"` in the `project.clj`.
4. Run `lein deps` to see if everything is ok.

The downside of this approach is that all necessary dependencies, even those of
Maven/Clojars repository, will be putted in the `repo` folder. So, in the first time
that you run  `lein deps` all the dependencies will be downloaded again. But, of
course, you don't need to add all of them to your version control repository. =)

