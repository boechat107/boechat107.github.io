---
layout: post
title: "Reading text from images"
description: "Gentle introduction to Optical Character Recognition (OCR)"
category: programming
tags: [image processing, OCR]
mathjax: true
lang: 
trans: 
---
{% include JB/setup %}

* toc
{:toc}

For the last 3 years I have been working with image processing and machine learning
models to be able to extract text from images.
And in many situations where I need to talk about the job, I use to have a feeling
that the listener can't really grasp the problem, it seems that some necessary 
common background is missing.
This is terrible when we need to discuss deadlines or to set prices on projects.

In this post, I'll try to give some basic understanding about the difference between
a *raw* text and a text represented by an image, then give an overview of a common 
approach to the problem and, finally, showing an example.

* Text localization, pre-processing, segmentation and recognition;
* Practical example: captcha or another text

## Text image and raw text

For a computer, there is a huge difference between a text and an image that contains
text. When we input characters of a text using the keyboard, for example, the
computer can easily *know* if we have inputted an "a" or a "k", because it has
specific representations for each symbol, representations for the signals sent by the
keyboard. In another hand, images of a text can have almost infinite different 
representations in the computer.

{% include image.html url="http://www.unselfishblowout.com/KillerText/images/TextExamples.jpg" description="Text can be represented by images in an almost infinity different ways." width="400" %}

A digital image is usually represented as a matrix of integer values in the
interval $[0, 255]$, which means that the image is just a composition of
unicolored small parts called *pixels*.
For gray scale images, zero represents black, 255 represents white and the
other values are shades of gray [^1]. 
[Color images](http://en.wikipedia.org/wiki/Color_image) are commonly
represented by three matrices: $R$ (red pixels values), $G$ (green pixels) and
$B$ (blue pixels), like a composition of three gray scale images.
Figure 2 shows three different image representations of the letter "e"; although
the pixels values are different in each matrix, humans can easily identify
the character.

[^1]: Binary images are gray scale images with just totally black and/or totally white pixels, which are usually represented with 0s and 1s instead of 0s and 255s.

{% include image.html url="http://annystudio.com/misc/anti-aliased-fonts-hurt/text-rendering-methods.gif" description="Three different representations of a letter: a binary, a gray scale and a color image." width="" %}

Although the computer *sees* images just like agglomerates of data, we can make it
learn to extract some meanings from them. We did learn to interpret images too.
The [human eyes](http://www.healthline.com/human-body-maps/optic-nerve)
works very similarly to a camera, sending electrical signals to the 
brain, which is responsible to interpret and to give them some meaning. But usually 
we are not aware of this process, our brains have learnt it very well when 
[we were just babies](http://en.wikipedia.org/wiki/Infant_vision).

So, how could we teach a computer to extract some meaning from an image? In the
case of texts, that is where *Optical Character Recognition* (OCR) comes in.

## How OCR works

Roughly speaking, the OCR process usually involves two different steps: text
detection/localization and character recognition. The first step is very important
when we want to extract text from natural images or when we don't even know if 
there really is any text in the image. Otherwise, for simplicity, we could focus 
on the second step, which is fundamental for any image that has text.

The "traditional" problem of the character recognition is to find patterns for each 
character class. Given the simplest image unit for an OCR system, the image 
of a single character (look at **figure**), this system needs to assign a 
character class accordingly to the pixels values.
This task would be easy if every image of an "A" or of a "5" were always the 
same, i.e., if their pixels values were always the same values. But any slight
difference of illumination, rotation angle or typeface could be enough to make
the pixels values look very different from other images of the same character 
class.

One possible approach to the problem is to map the pixels values into points in 
a feature space.
With each character image represented by a point in the feature space, the OCR 
system could try to find patterns in those points' positions and their respective
character class. 
Then, each new character image would be mapped into the same 
feature space and the most similar pattern would be matched, giving the most likely
character class of the image.

Perhaps the most obvious feature space is that composed of the own pixels values, 
where the number of pixels gives the dimension of this space. For example, for images
of size 4x4, the feature space would be $\mathbb{N}_{<256}^{16}$, a sixteen dimensional
space where each dimension could assume an integer value in the interval $[0, 255]$.
A whole image would be represented as a point in that space.

Considering that fact, we could think about a simple way to compare images without
thinking of exactly equality of pixels values: the Euclidean distance between images.
Now known, labeled images $x_l^i$ (character class $l$, known sample $i$) can
be used to guess an unlabeled image $x$ by measuring the Euclidean distance
$d_l^i(x_l^i, x)$ and taking the label $l$ of the smallest $d_l^i$.
In another words, we are taking the class of the nearest neighbor [^2] of $x$ in the 
feature space.

[^2]: This is the basic idea of a very common machine learning algorithm called [k-Nearest Neibors](http://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm).

## OCR trends

Probably the hottest research fields in machine learning today, the application of
[Deep Neural Networks](http://devblogs.nvidia.com/parallelforall/accelerate-machine-learning-cudnn-deep-neural-network-library/).
to image processing has moved the 
[*image understanding*](http://googleresearch.blogspot.com.br/2014/09/building-deeper-understanding-of-images.html)
to another level.
As its own name says, DNNs are similar to normal neural networks, but they are
deeper because of their bigger number of hidden layers. 

In the OCR field, DNNs has been used to recognize better text in natural images or 
photographs, where the visual appearance of text has a wider variability than 
scanned documents.

One very interesting application was showed in 
[this paper](http://arxiv.org/abs/1312.6082) from a Google's research team.
There a DNN is used to recognize a whole word (or a sequence of numbers) at once,
i.e., the network recognizes the characters without any explicity text segmentation,
without isolating single characters.
[Recaptcha images](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQWV3rmKq-EmnZBD_UeLhbAQaPWeuoXWPUCTC7mKtfwztiWcb8G),
that are usually very hard to be segmented into single characters, were *easily* 
recognized by such DNN.
But, although these networks are very powerful, their training is usually harder
than the training of other popular ML techniques.
I hope to explore some of theirs details in another post.

## Conclusion

## References

* http://en.wikipedia.org/wiki/Optical_character_recognition
* http://www.explainthatstuff.com/how-ocr-works.html
* http://www.andrewt.net/blog/posts/how-to-crack-captchas/
* http://en.wikipedia.org/wiki/Infant_vision
* http://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm
* http://devblogs.nvidia.com/parallelforall/accelerate-machine-learning-cudnn-deep-neural-network-library/
