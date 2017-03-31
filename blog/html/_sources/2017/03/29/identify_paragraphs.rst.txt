Identify Paragraphs
===================

I'm learning computer vision and I'm using the `Opencv <http://docs.opencv.org/>`_
and `Python <https://www.python.org/>`_ to write algorithms in this field of computer science. This field
is very interisting because it envolves computer science and math. It is necessary to have good ideas to solve the problems.

Here, I will show my first algorithm in computer vision. It is to identifies paragraphs in a text image. It consider that the image
have a white background and that the characters have a black color.

This algorithm have two main ideas: First, it removes blank lines and columns before finding the text. After this, it calculate
the number of blank lines between no blank lines. The minimum and the maximum number of blank lines is computed. Though
that the number of blank lines between two no blank lines is greather or equal 90%(maximum - minimum), we consider that we found
a new paragraph.

Below, you can see the image with text to input in the algorithm:

.. figure:: input.png
    :alt: Text extract from python's doc

And in the image below is possible to see the image with the identified paragraphs:

.. figure:: output.png
    :alt: Identified paragraphs

You can download the algorithm and get more details in the
repository: `https://github.com/joaojunior/identify_paragraphs_in_image <https://github.com/joaojunior/identify_paragraphs_in_image>`_

.. author:: default
.. categories:: none
.. tags:: none
.. comments::
