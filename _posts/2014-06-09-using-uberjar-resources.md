---
layout: post
title: "Using uberjar resources"
description: "How to use file resources encapsulated in the jar"
category: Programming
tags: [Clojure, Java]
---
{% include JB/setup %}

This post concerns the problem of using a resource file encapsulated in the jar with the
source code files. 

In my case, I had a third party function
that expected a `BufferedReader`, which should have the data of my resource file located
inside the jar. The solution for my problem came from this 
[post](http://stackoverflow.com/a/2271952/747872) and the Clojure code version follows
below:

{% highlight clojure %}
(defn get-uberjar-resource
  "Finds the resource inside the jar and returns a BufferedReader object. The
  resource path is the expected path inside the jar."
  [path-str]
  (-> (Thread/currentThread)
      (.getContextClassLoader)
      (.getResourceAsStream path-str)
      (InputStreamReader.)
      (BufferedReader.)))
{% endhighlight %}

Note that `clojure.java.io/resource` returns the URL of the desired
resource, which can't be handle as a `File`.
