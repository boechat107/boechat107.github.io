---
layout: post
title: "My First Stupid Mobile App"
description: "The first experience we never forget..."
category: Programming
tags: [Clojure, Mobile, Clojurescript]
mathjax:
lang:
trans:
---
{% include JB/setup %}

<img src="http://witzzer.com/wp-content/uploads/2016/09/10-things-only-really-stupid-people-will-understand.png" width="600">

Everybody seems to have developed a mobile app in his life, so I always thought
that mine was just a matter of time. Last week I finally decided to take this
next step and I started to look for more information about mobile
development.

Around three years ago, I tried to play with the Android SDK and
Java, but I found the documentation very confusing and I thought I didn't have
the stoma... I wasn't mature enough to handle the complexity just using my spare
time.
From that time to now, I have been hearing a lot about
[hybrid apps](https://developer.salesforce.com/page/Native,_HTML5,_or_Hybrid:_Understanding_Your_Mobile_Application_Development_Options)
and how useful they can be for small teams and where performance is not a
requirement.

I was decided to build my first app using [Cordova](https://cordova.apache.org/)
and, if possible, [Clojurescript](https://clojurescript.org/), and I started to
search for related documentation and experiences of other developers.
I don't remember exactly how I ended up reading something
about [React Native](https://facebook.github.io/react-native/), but it got my
attention, since I had some happy experiences with
[Reagent](http://reagent-project.github.io/).
In addition, with React Native I would be able to generate a native app (with
[potential performance issues](https://news.ycombinator.com/item?id=11865483)).

I thought: "if it is Javascript and React, I could probably use Clojurescript
and Reagent for a quick test and have some fun at the same time". I was excited!
I could replace Java (at least for a small experiment) with a functional
approach and much more fun! So, I changed my focus to Clojurescript with React
Native and I found a
[blog post](https://medium.com/@tiensonqin/my-experience-with-clojurescript-and-react-native-81bb1d3bb2c4#.cf9m972km)
about an app called [Lymchat](http://lymchat.com/) and the
projects [Natal](https://github.com/dmotz/natal)
and [Re-Natal](https://github.com/drapanjanas/re-natal).
Once again, I naturally moved my focus to Re-Natal.

Re-Natal is based on [re-frame](https://github.com/Day8/re-frame), which was new
to me. I knew about the project before because of
[their Reagent tutorials](https://github.com/Day8/re-frame/wiki) I used when I
implemented my first reactive web applications, but, as I didn't want to start by
using a framework, I didn't dig into it.
By using Re-Natal, I decided to read the re-frame docs to have an opinion. I got
surprised by their functional approach! It was really nice to understand it, I
felt like learning more about functional programming in general, not just a framework.

Now I felt ready to start playing with some code and I generated a project
using Re-Natal. To install all the required dependencies, I followed
[a post from gadfly](https://gadfly361.github.io/gadfly-blog/2016-11-13-clean-install-of-ubuntu-to-re-natal.html)
and everything worked almost as expected.
I didn't installed [Watchman](https://facebook.github.io/watchman/) (since it
was supposed to be
[optional](https://facebook.github.io/react-native/docs/getting-started.html#content))
and I didn't want to use
[Genymotion](https://www.genymotion.com/), I preferred to try the Android
Virtual Device.

When starting an AVD, selecting the option "use host GPU" makes the emulation
much faster and nicer to work with a REPL, but I got
an
[error](http://stackoverflow.com/questions/35911302/cannot-launch-emulator-on-linux-ubuntu-15-10):

```
libGL error: unable to load driver ...
...
```

Setting the environment variable `ANDROID_EMULATOR_USE_SYSTEM_LIBS`, as described
in [this answer](http://stackoverflow.com/a/36625175/747872), was enough to
solve the problem.

Another error that happened was a `ENOSPC` (not enough space) error. This weird
problem was solved by [this answer](http://stackoverflow.com/a/32600959/747872).
Maybe using Watchman would have avoided this error, but I didn't tried.

Finally, after just a few tweaks in the generated code, I was able to have my
first mobile app! After trying the Java SDK (and feeling stupid for not getting
it quickly), this implementation looked so simple, so easy, so... stupidly easy
(the final feeling was almost the same, but for different reasons).
The result was this simple Android project:
[Countdown](https://github.com/boechat107/countdown-cljs).
