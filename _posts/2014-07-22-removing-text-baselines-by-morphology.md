---
layout: post
title: "Removing text baselines by morphological operations"
description: "Using mathematical morphology (erode/dilation) to remove text baselines"
category: Research Problems
lang:
trans: 
tags: [Image Processing, OCR]
---
{% include JB/setup %}

Some weeks ago, I was developing a program to extract some specific informations from
a photographed document, which is composed of many fields surrounded by layout lines.
Most of the time those layout lines didn't bothered me, but then I started finding a lot
of sample images where the characters were touching the lines or were very very close
to them, messing up with my text line segmentation strategy. 

As I'm always trying to seek similarities between my problems and academical researches,
I stumbled on an interesting [paper](http://www.ee.bgu.ac.il/~dinstein/stip2002/Seminar_papers/Hershkovitz_Extraction%20of%20bankcheck.pdf)
that uses very simple procedures to remove handwritten text baselines. The technique
described there uses mathematical morphology operations (basically, a sequence of
[dilations](http://en.wikipedia.org/wiki/Dilation_(morphology)) and
[erosions](http://en.wikipedia.org/wiki/Erosion_(morphology))) on gray scale level,
claiming that it avoids loosing too much information and generates smoother boundaries
than that in binary images.

Let's test it using small scripts in [Octave](http://www.gnu.org/software/octave/):

{% highlight matlab%}
img = imread('handwritten1.jpg' );
gray = rgb2gray(img);
h_gray = baseline_removal(gray, 30, 3);
imshow(h_gray)
subplot(1,2,1), imshow(gray)
subplot(1,2,2), imshow(h_gray)
{% endhighlight %}

<!---
![text baselines](http://i59.tinypic.com/2mnoow0.png)
<img alt="text baselines" src="http://i59.tinypic.com/2mnoow0.png" width="700">
-->

{% include image.html url="http://i59.tinypic.com/2mnoow0.png" description="Baseline removal's result. The original image can be found at http://handwritinguniversity.com/images/dhembree/dannyhembree1.jpg." width="700" %}

For me, the results are pretty impressive! Although some small parts of the lines 
were not totally removed, the text seems intact, and I didn't even tunned the 
algorithm's parameters.

My implementation of the technique, the function `baseline_removal` (that can be found 
in one of my [github repositories](https://github.com/boechat107/imgproc_scripts)), has 
3 parameters: the grayscale image, the length of the shortest baseline feature to be 
eliminated and the height of the thickest part of the baselines. So, in the code above,
the minimal length chosen was 30 and the height was 3, without any detailed analysis of
the image, I just put some reasonable values.

This `baseline_removal` function has an optional parameter that was not used here. Its 
last and optional parameter is the angle from X-axis to X-Y projection of the line and 
it allows the removal of straight lines in any direction. As its default value is zero,
we don't need to worry about it for horizontal lines. If one wants to remove vertical
lines, for example, the value 90 should be passed.

## Reference

Ye, X., Cheriet, M., Suen, C., & Liu, K. (1999). Extraction of bankcheck items by mathematical morphology. International Journal on Document …, 2(2-3), 53–66. doi:10.1007/s100320050037
