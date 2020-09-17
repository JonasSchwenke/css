---
layout: post
title: Webscraping Text from Images with Python
date: 2020-09-15 13:32:20 +0300
description: Find out how to convert text displayed on an image into a string with optical character recognition # Add post description (optional)
img: webscraping_images.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Python, OCR, images, text]
---
Social Scientists often work with text data. But what if the desired text is part of an image and hence not directly scrapable with ```Selenium``` or ```BeautifulSoup```? 
In that case Optical Character Recognition (OCR) is needed to convert the image text into a string. In this example I will show you how to do that with the little help of a couple of Python packages.

## Example: Facebook Posts

Let's take the popular German Facebook page [Faktastisch](facebook.com/faktastisch) which posts interesting (and sometimes ridiculous) facts like this one:


![Image with text]({{site.baseurl}}/assets/img/faktastisch.jpg){: .center-image}

English:
> Kim Jong-un travels with his own mobile toilet. This is to prevent his excrements from falling into the wrong hands, as they contain information about his health.

To access this image in Python I used the [facebook-scraper package](pypi.org/project/facebook-scraper/) which returns all meta information about the scraped posts, including any image [URLs](https://scontent-zrh1-1.xx.fbcdn.net/v/t1.0-9/119551251_3859069730824010_1525164325381591588_o.jpg?_nc_cat=105&_nc_sid=730e14&_nc_ohc=i1l_QNfIiMkAX-mjbXc&_nc_ht=scontent-zrh1-1.xx&oh=962d8220995ab6f232386ecee2268608&oe=5F889D5E).

As soon as you have your URL (wherever it may come from) you can use the following code to read the image into memory as a numpy array.
For that you may need to install [scikit-image (skimage)](https://pypi.org/project/scikit-image/). You will also need matplotlib for plotting.

```python
from skimage import io
from matplotlib import pyplot as plt

url = 'INSERT_YOUR_URL'

# read image into memory
img = io.imread(url)

# plot image
plt.imshow(img);
```

By plotting the image you can now easily identify the desired section of the image and crop it accordingly with simple index slicing.


```python
img_cropped = img[500:1500, 0:2048]

plt.imshow(img_cropped);
```

![Cropping image]({{site.baseurl}}/assets/img/cropping.jpg){: .center-image .smaller-image-width}

Now that we have the desired text section of the image we need to apply Optical Character Recognition (OCR). The best package for this task is [pytesseract](https://pypi.org/project/pytesseract/), a Python wrapper for Tesseract, which also serves Google Books. Because this example is in German, you may need to download the language data set via ```brew install tesseract-lang```.

```python
import pytesseract

print(pytesseract.image_to_string(img_cropped, lang='deu'))
```

![OCR result after cropping]({{site.baseurl}}/assets/img/cropping_ocr_result.jpg){: .center-image .image-width-400}

Via the ```image_to_string``` function and by specifying the desired language, pytesseract performs OCR for German on the cropped image.
However, the results are rather discouraging.
Only the bottom part of the text can be identified as letters by pytesseract. This is because the other text is not easily distinguishable from the background noise.

## Improving OCR Accuracy with Thresholding

Luckily, there is a neat image manipulation trick for this sort of problem: Thresholding. This process basically turns all pixels that exceed a given value into another given value. The [cv2 package](https://pypi.org/project/opencv-python/) has five different types of thresholding and good explanations of their functioning [here](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_thresholding/py_thresholding.html). 

In this example the text is white, so we want all pixels, that are not perfectly white to turn into black to get a uniform background for the text. Then the whole thing can be inversed for clear, black-on-white text. Here you might have to experiment a little with the values given to the ```cv2.threshold``` function. After a little back and forth of applying thresholding and plotting, I found these values to work perfectly for our example.

```python
import cv2

# apply thresholding
ret,thresh2 = cv2.threshold(img_cropped,200,255,cv2.THRESH_BINARY_INV)
```

![OCR result after thresholding]({{site.baseurl}}/assets/img/cropping_ocr_result.jpg){: .center-image .image-width-400}

After performing OCR with pytesseract on the preprocessed image, the resulting string is a perfect representation of the text. It can now be used for further analysis or to create a dataset.
Depending on the images you have to work with, other preprocessing steps might be necessary. To move further into that topic, I recommend this article:

- [OCR with Python, OpenCV and PyTesseract](https://medium.com/@jaafarbenabderrazak.info/ocr-with-tesseract-opencv-and-python-d2c4ec097866)

Please let me know about any errors or improvements in the comment section below!
