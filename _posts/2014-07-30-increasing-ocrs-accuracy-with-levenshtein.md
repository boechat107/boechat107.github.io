---
layout: single
title: "Increasing OCR's accuracy with the Levenshtein distance"
description: "Using the Levenshtein distance to increase the OCR's accuracy"
category: programming
tags: [Image Processing, OCR, Spell Checking]
mathjax: 
lang: en
trans: /melhorando-ocr-usando-a-distncia-levenshtein/
---


## Problem

Almost one month ago, I was improving the code of my OCR program and thinking
about some wrongly classified characters I was seeing on the screen.
The program should process photographs of some specific documents composed of
many fields, where one of them contains a number representing a year.
I was frustrated because many times some characters of this number were 
incorrectly classified, but I, a trained monkey, could easily guess the 
correct number. Some examples:

* 2509
* 0012
* 2097

OK, guessing could not lead to the true number, the last two numbers could be wrongly
classified, but it could give me a much better result than my OCR program alone.

## The monkey idea

At first, I thought about creating another SVM model. I knew that the current model
was not good, it was made quickly, with artificial samples. I could improve it by
using carefully selected real samples, for example, but it would cost me some monkey
hours.
Then I thought about another interesting idea, one that could force me to learn 
something new: a spell corrector. I Googled it.

The most interesting result was the famous (I realized) 
[article](http://norvig.com/spell-correct.html) written by Peter Norvig. Excellent
article, but too much for the problem. However, the "edit distance" idea was not
new for me and I remembered of the 
[Levenshtein distance](http://en.wikipedia.org/wiki/Levenshtein_distance). It
seemed faster to test, since it has countless implementations over the web.
Then I wrote this following simple code in Clojure:

{% highlight clojure %}
(ns cetip.spell-corrector
  (:require 
    [incanter.stats :refer [levenshtein-distance]]))

(defn levenshtein-similar
  "Returns the dic's word most similar to target calculating the Levenshtein 
  distance between them. The dictionary dic is just a sequence of words."
  [target dic]
  {:pre [(string? target)]}
  (apply min-key (partial levenshtein-distance target) dic))
{% endhighlight %}

The function `levenshtein-similar` just calculates the Levenshtein distance 
between the target word (the word to be corrected) and a sequence of possible 
words and takes the possible word with the minimum distance.

Now we can test the code for the examples above:

{% highlight clojure %}
=> (levenshtein-similar "2509" (map str (range 1998 2015)))
2009
=> (levenshtein-similar "0012" (map str (range 1998 2015)))
2012
=> (levenshtein-similar "2097" (map str (range 1998 2015)))
2007
{% endhighlight %}

In my case, it was easy to assume a range of possible numbers. I used that piece of
code in my program and the OCR's accuracy was improved a lot for my test images.

## Conclusion

The implemented solution is far from the best, I know. However, it saved me time and
helped me to see another possible approach (spell correction) to improve the accuracy
of my OCR system. Perhaps more improvements can be made on another fields whose
values can be part of a small set of words.
