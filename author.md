---
layout: page
title: "About me"
description: "About me"
permalink: /author/
---
{% include JB/setup %}

<img src="/images/avatar5.jpg" width="150" align="right">

Interests (without a specific order):

* Functional Programming, Machine learning, Web Applications
* Basketball, soccer, weight lifting
* Good books and movies (I'm always open to good suggestions)
* Interesting conversations (my email is at the bottom of the page :-)
* Open-source projects
* Libertarianism

Mimicking my friend [patito](http://patito.github.io/), my CV as code (or Clojure):

{% highlight clojure %}
{:experiences
 (->> [[:brick-abode ["Full-stack Software Engineer"
                      "6 months"
                      ["Python" "Postgres"]]]
       [:iter ["Full-stack Software Engineer"
               "11 months"
               ["Clojure" "MySQL" "MongoDB"]]]
       [:neoway ["Software Engineer, R&D"
                 "3 years and 8 months"
                 ["Java" "Python" "Clojure"
                  "Caffe" "BoofCV" "Scrapy"
                  "Postgres" "Redis" "RabbitMQ"]]]]
      (map (fn [[c [title time techs]]]
             [c {:title title :time time :technologies techs}]))
      (into {}))
 :open-source-contributions
 (map (fn [[t tp u]] {:title t :type tp :url u})
      [["scikit-learn" [:doc :code] "http://scikit-learn.org/"]
       ["luminus-template" [:code] "http://www.luminusweb.net/"]
       ["caffe" [:doc] "http://caffe.berkeleyvision.org/"]
       ["monger" [:code] "http://clojuremongodb.info/"]
       ["expiring-map" [:code] "https://github.com/luminus-framework/expiring-map"]
       ["ring-mock" [:code] "https://github.com/ring-clojure/ring-mock"]
       ["svm-clj" [:code] "https://github.com/r0man/svm-clj"]])
 :open-source-projects
 (map (fn [[t u]] {:title t :url u})
      [["ring-ttl-sessions" "https://github.com/boechat107/ring-ttl-session"]
       ["eye-boof" "https://github.com/boechat107/eye-boof"]])}
{% endhighlight %}

Some other public information:

* [Github repositories](https://github.com/boechat107)
* [Gists](https://gist.github.com/boechat107)
* [List of books](http://www.goodreads.com/boechat107) I'm reading or that I have
read.

- - -

> "In God we trust, all others bring data."

--- *William Edwards Deming* <br>
A quote from the book "The Elements of Statistical Learning".
