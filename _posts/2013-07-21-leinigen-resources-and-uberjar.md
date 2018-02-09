---
layout: single
title: "Leinigen resources and uberjar"
description: "Problem concerning the copy of a resource file."
category: Programming
tags: [Leinigen, Clojure]
---


Problem concerning the copy of a resource file.

{% highlight clojure %}
(:require
    [clojure.java.io :as io])
(:import 
    [org.apache.commons.io FileUtils])

(defn copy-template
  "Copy the template file to the same directory of the given filepath and rename it
  with the same name of the given filepath, but using XLS extension."
  [fpath]
  (let [template (log/spy (io/resource "template.xls"))
        out-path (s/replace fpath #"\.\w+$" ".xls")] 
    (FileUtils/copyURLToFile template (io/file out-path))
    out-path))
{% endhighlight %}
