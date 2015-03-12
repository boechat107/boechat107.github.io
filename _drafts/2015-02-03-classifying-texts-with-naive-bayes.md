---
layout: post
title: "Classifying texts with Naive Bayes"
description: "How to classify texts based on their words."
category: Machine Learning
tags: [natural language processing]
mathjax: true
lang: 
trans: 
---
{% include JB/setup %}

* toc
{:toc}

## Problem description

Some weeks ago, a colleague came to ask me an idea to help him classify building 
constructions by their description. 
He have found that usually some keywords are good indicators of the building class,
but there were too much data to analyse.
Although I haven't seen the data, the brief description of the problem made me think 
that this case could be a good application for a Naive Bayes Classifier.

In this post, I describe my intuition about the Naive Bayes Classifier and it's 
mathematical formulation. Then, I present a simple example of usage of this 
classifier using a Python library,
[TextBlob](https://textblob.readthedocs.org/en/latest/#).

* The necessity of having manually labeled texts (in this case, it was done by 
the people involved).
* Non-mathematical idea of the algorithm, basic assumptions (naive), explanation
of the equation 13.4.
* Mathematical formulation.
* Python libraries
* Code example

## Naive Bayes Intuition

Our main objective here is to classify a text into different categories 
$c \in \mathbb{C}$ depending on its vector of words $X = [x_1, ..., x_n]$.
Here we already have to make a modeling decision: to consider all occurrences
of each word (multinomial model) or just to mark the presence or absence of each 
one (Bernoulli model). Let's take the former approach. 

As we are going to construct the model from a limited amount of data (even 
a million of texts can be considered limited), we can expect to end up with 
a solution that gives us a probability of a text $d$ to belong to a class $c$:
$P(c|d)$.

Without entering into the mathematical details, we can make some conjectures
about this probability:

* It depends on the frequency of a document class. Frequent classes in our 
documents database may be a good indicator for the class of new documents.
* Obviously, the occurrence of words in a document can be crucial to
correctly assign its class. But we don't know exactly which words are 
important or how much they are.
* The occurrence of a word is probably related to the occurrence of another 
word in the text. But, perhaps, we can have satisfactory results even 
ignoring this fact (which is a *naive* assumption). We could assume that the
occurrence of a word doesn't depend of other words.

Those conjectures try to measure how much evidence we have in a document 
to classify it into a specific category, considering our known documents 
database. In the end, we will take the class with the highest probability,
or:

$$
c_{\mbox{answer}} = \arg \max_{c \in \mathbb{C}} P(c|d)
$$

## Mathematical formulation 

Let's start by expanding $P(c|d)$
using the Bayes' theorem.

### Bayes rule and conditional probability

$$
P(c|d) = \frac{P(d|c) P(c)}{P(d)}
$$

$P(d)$
depends on the occurrences of the words 
$x$; 
it could be calculated from our documents database and seen as a constant
value. So, we could just ignore this term and try to calculate 
$\hat{P}(c|d)$:

$$
\begin{equation}
P(c|d) \propto \hat{P}(c|d) = \hat{P}(d|c) \hat{P}(c)
\label{main}
\end{equation}
$$

where $\hat{P}$ comes from the fact that we don't know the true values
of those probabilities, but estimate than from our documents database.
Now, considering the chain rule of repeated applications of conditional
probability 

$$
P(a,b,c,d) = P(a) P(b,c,d|a) = P(a) P(b|a) P(c,d|a,b)
$$

we can derive 
$\hat{P}(d|c)$ as 

$$
\begin{align*}
\hat{P}(d|c) = \hat{P}(X|c) &= \hat{P}(x_1|c) \hat{P}(x_2, ..., x_n|c,x_1) \\
         &= \hat{P}(x_1|c) \hat{P}(x_2|c,x_1) \hat{P}(x_3, ..., x_n|c,x_1, x_2) \\
\end{align*}
$$

Following our previous conjectures and assuming our **naive** conditional 
independence among the words $x$, we can write our final definition
of $\hat{P}(d|c)$:

$$
\hat{P}(d|c) = \hat{P}(X|c) = \prod_{i=1}^n \hat{P}(x_i | c)
$$

### Frequency of document classes

Returning to our target, the probability of a document $d$ being in the 
class $c$ given in the equation \eqref{main}, 

$$
\begin{equation}
\hat{P}(c|d) = \hat{P}(d|c) \hat{P}(c) = \hat{P}(c) \prod_{i=1}^n \hat{P}(x_i | c)
\label{target}
\end{equation}
$$

we need to calculate the frequency of the document classes $P(c)$ from our documents
database.
One simple and intuitive way to do this is 

$$
P(c) = \frac{N_c}{N}
$$

where $N_c$ is the number of documents in the class $c$ and $N$ is the total number 
of documents in our documents database.

### Influence of each word

The last missing piece is the calculation of $\hat{P}(x_i | c)$,
the probability of a word $x_i$ occurring in a document of a class $c$.
The approach can be very similar to the classes' frequency:

$$
\begin{equation}
P(x_i|c) = \frac{N_{x_i, \, c}}{N_{x, \, c}}
\end{equation}
$$

where $$N_{x_i,c}$$ is the number of occurrences of $x_i$ in the class $c$; and
$N_{x,c}$ is total number of words in the class $c$.
These numbers could be easily taken if we group all texts of a class 
in one single text blob.

However, a practical problem arises.
Imagine that we are working on a text classifier for documents about sports and 
the classes are just different sports, like texts about basketball, soccer or 
baseball.
Now, suppose that in our documents database there is no occurrence of the word 
"cancer" in a text about soccer.
What happens with our classifier when it faces a new text containing the word 
"cancer"? 

As we can see in equation \eqref{target}, the probability the document above 
being in the class soccer is zero because $$P(\mbox{cancer}\,|\,\mbox{soccer})=0$$,
even if the document is clearly a text about soccer.

## References

* Christopher D. Manning, Prabhakar Raghavan, and Hinrich Schütze. 2008.
[*Introduction to Information Retrieval*](http://nlp.stanford.edu/IR-book/).
Cambridge University Press, New York, NY, USA. 
* [http://en.wikipedia.org/wiki/Naive_Bayes_classifier](http://en.wikipedia.org/wiki/Naive_Bayes_classifier)
