---
layout: single
title: "My first experience with Docker"
description: "How I found my first useful application"
category: programming
tags: [Virtual Machine, Programming Environment]
mathjax: 
lang: 
trans: 
---


Sacrificing accuracy for simplicity, I would say that [Docker](http://www.docker.com/)
is a platform to run applications as virtual machines in a very lightweight way.
As I didn't get at a first glance (it is a little complicated for me yet), perhaps
my first Docker experience can help you to start using it quicker than me.

## Problem

I needed a R programming environment, which I didn't have at that moment.

## Docker solution

```sh
$ docker pull docker-debian-r
```

```
$ docker run -t -i eddelbuettel/docker-debian-r:add-r R
```

```
$ docker run -t -i -v $(pwd):/current_dir eddelbuettel/docker-debian-r:add-r R
```

Install packages and then commit the modifications:

```
docker commit -m "Some additional R packages for the Coursera's Data Science class"\
 hungry\_heisenberg boechat107/debian-r-programming
```

Push the modifications.

## Conclusion
