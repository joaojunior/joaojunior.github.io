---
title: "Identifying Paragraphs"
date: 2017-03-29T17:00:19-05:00
draft: false
tags: ["Computer Vision", "OpenCV", "Python"]
---

I'm learning computer vision and I'm using the [Opencv](http://docs.opencv.org/)
and [Python](https://www.python.org/) to write algorithms in this field of computer science. This field
is very interesting because it involves computer science and math and also, it is necessary to have good ideas to solve problems.

Here, I will show my first algorithm in computer vision. It is to identifies paragraphs in a text image. It consider that the image
have a white background and that the characters have a black color.

This algorithm have two main ideas: First, it removes blank lines and columns before finding the text. After this, it calculate
the number of blank lines between no blank lines. The minimum and the maximum number of blank lines is computed. Though
that the number of blank lines between two no blank lines is greather or equal 90%(maximum - minimum), we consider that we found
a new paragraph.

Below, you can see an image with text to input in the algorithm:
![Text extract from python's doc](/identify_paragraphs/input.png)

And in the image below is possible to see the image with the identified paragraphs:

![Identified paragraphs](/identify_paragraphs/output.png)

You can download the algorithm and get more details [here](https://github.com/joaojunior/identify_paragraphs_in_image)
