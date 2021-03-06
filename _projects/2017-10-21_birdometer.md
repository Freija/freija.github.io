---
layout: default
title: "Birdometer"
date: 2017-10-21
last_update: 2017-10-21
thumbnail: "birdometer/2017-09-21_16.06.01_9.png"
primary_image: "birdometer/labelling-example3.png"
github_link: "https://github.com/Freija/birdometer-ml"
super_short_description: "A project to capture and classify images of hummingbirds."
short_description: "A hummingbird feeder instrumented with a Raspberry Pi captures images of hummingbirds. These images are processed by a machine learning pipeline."
long_description: "A hummingbird feeder instrumented with a Raspberry Pi captures images of hummingbirds. These images are processed by a machine learning pipeline. The first step is to classify each image according to if there is indeed a bird in the image or not. For this, a training set of about 3800 images were labeled and are being used to test multiple classifiers."
tags:
- docker
- sklearn
- raspberry pi
- machine learning
- image classification
- birdometer
---
# Birdometer
### Capturing and classifying hummingbird pictures
<ul class="tags">
  {% for tag in page.tags %}
    <li><a href="/tags#{{ tag }}" class="tag">{{ tag }}</a></li>
  {% endfor %}
</ul>
[https://github.com/Freija/birdometer](https://github.com/Freija/birdometer)

1. [Introduction](#introduction)
  * [The local hummingbirds](#Locals)
  * [What is the Birdometer](#Whatisbirdometer)
  * [Goals](#Goals)
  * [Challenges](#Challenges)
2. [Image processing](#Components)
  * [Creating and understanding the labeled data-set](#Labeling)
  * [Image preprocessing](#Preprocessing)
3. [Image classification](#Classification)
  * [Overview](#Overview)
4. [Current status](#Deployment)
5. [Future ideas](#Future)

<div id='Introduction'/>
## Introduction
<div id='Locals'/>
### The local hummingbirds
Hummingbirds are pretty amazing. I am lucky enough to live in an area where they are common. The local resident is the Anna's Hummingbird, but we can also find some Allen's/Rufous Hummingbirds (notoriously hard to distinguish) for certain months of the year.
<p>
<img src="{{ site.baseurl }}/images/birdometer/img_4969.jpg" alt="Anna's hummingbird 1" style=" width: 49.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/20170613_0098.jpg" alt="Anna's hummingbird 2" style=" width: 49.5%;"/>
<em><small> Anna's Hummingbird (left) and Allen's/Rufous Hummingbird (right). These pictures were taken with a DSLR camera, not the Birdometer. Copyright Freija Descamps. All rights reserved.</small></em>
</p>
These hummingbirds are very small (7.6-10.9cm) and only weigh a few grammes. They live from drinking nectar from flowers and eating the occasional small bug, which they catch in flight.
<div id='Whatisbirdometer'/>
### What is the Birdometer
The Birdometer is a hummingbird feeder that I instrumented with a Raspberry Pi board with a connected Raspberry Pi camera and IR sensor. The Birdometer is currently set up to take 10 images whenever the IR sensor triggers. The Raspberry Pi code runs in a Docker. The code can be found here:
1. Raspberry Pi: [https://github.com/Freija/birdometer-rpi](https://github.com/Freija/birdometer-rpi)

2. Image analysis with Python: [https://github.com/Freija/birdometer-ml](https://github.com/Freija/birdometer-ml)

The camera captures a top view of the birds as they are approaching and feeding. The camera alignment is done by aligning two marks: one on the feeder and one on the Birdometer. This alignment is not perfect and the location of the perch and feeding hole can change slightly from image to image. The orientation of the feeder itself does change compared to the ground, which means that the background can be different between different sets of images.
<p>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-21_16.06.01_9.jpg" alt="example birdometer 1" style=" width: 49.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-20_18.59.19_2.jpg" alt="example birdometer 2" style=" width: 49.5%;"/>
<em><small> Two example Birdometer images showing two different birds, male (left) and female/juvenile (right) Anna's Hummingbird. </small></em>
</p>

<div id='Goals'/>
### Goals
The first goal is to use the data to monitor the visits of the hummingbirds to the feeder, basically to record the time of each visit. This will allow us to potentially answer some interesting questions, like for example:

 * How often does a bird visit?

 * What times of the day are most popular?

 * Is there a correlation with temperature? Sun/rain/wind?

 * Is is possible to predict the number of visits for a particular day or other time metric?

The data will be displayed on a website (to be constructed) containing a bird time-line showing the time of each visit and also the associated images for each visit. Eventually, it would be cool to try to identify individual birds, see discussion of future ideas [here](#Future).

<div id='Challenges'/>
### Challenges
This section discusses some of the challenges that are currently being tackled or investigated.

<div id='false'/>
**False triggers**<br>
One of the initial challenge is to deal with the times that the IR sensor triggers without there being a bird in the camera field of view. I will call these ```false triggers``` and they lead to bird-less images. There are three types of (indistinguishable for each individual picture) false triggers:

 1. The IR sensor is triggered by something other than a bird.

 2. The IR sensor is triggered by a bird which stays outside the field of view of the camera.

 3. The IR sensor is triggered by a bird which leaves too fast or enters too slow to be captured by the camera.

So clearly, we need to implement an image classification to tag images: bird or no bird. Then, only images with birds will be used in the final data-analysis and visualization. In fact, ideally, images without birds should not be saved to disk.

<p>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-21_16.06.58_0.jpg" alt="beak peek 1" style=" width: 49.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-22_17.34.58_9.jpg" alt="beak peek 2" style=" width: 49.5%;"/>
<em><small> Two example Birdometer images showing just the beaks of the visiting hummingbirds entering the image. Can you find the beak in the picture on the right? It is at the bottom close to the center. Should these images be classified as having a bird or not? Also note the differences in orientation and background between the two images.</small></em>
</p>

**False triggers/missed birds trade-off** <br>
One way to deal with the false triggers is to limit the field of view of the IR sensor itself. I have experimented with different shapes (cones, tubes, etc) that limit the sensor's opening angle. This effectively reduces the false triggers. However, after some observation and testing, it became clear that this would lead to missed birds: occasions when a bird succeeds in feeding without triggering the sensor! That is much worse, in my opinion, than having to deal with a large number of false triggers.

Once we have the image classifier deployed on the Raspberry Pi, we can go to a continuous image-recording mode, only saving the images with birds to disk. The IR sensor signal can then serve as a tag rather than trigger.  

**Sun/shade differences** <br>
The feeder itself is located in a sunny spot of the garden. This causes the light to be very different at different times of the day. The hope is that it is possible to develop a classifier that is indifferent to this, but it is definitely something to keep in mind. It is possible to move the feeder or add shade to make the images more homogeneous if that turns out to be necessary.

**Background differences** <br>
The orientation of the feeder relative to the ground is not controlled for. This means that the background can look very different between different set of images. If this turns out to be an issue, a plate can be mounted on the bottom of the feeder to provide a homogeneous and constant background. However, this will add weight and will make the feeder more susceptible to wind.

<p>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-21_18.24.31_9.jpg" alt="background 1" style=" width: 49.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-21_20.23.08_1.jpg" alt="background 2" style=" width: 49.5%;"/>
<em><small> Two example Birdometer images illustrating differences in shade and background orientation. </small></em>
</p>

**Hardware**<br>
Currently, the Birdometer is powered through the Micro USB port using the standard adapter, plugged into an extension cord. This means that the unit is not wireless and I cannot currently leave it out overnight because of that. I'm investigating powering the unit through a combination of solar and battery power.
Also, the current prototype is not watertight and the raining season is coming up. I think it would be interesting to see the influence of rain and humidity on the feeding frequency. Therefore, I am designing a better electronics enclosure.


<div id='Components'/>
## Image processing
<div id='Labeling'/>
### Creating and understanding the labeled data-set
The first step towards creating the classifier (bird/nobird) is to manually label a set of images. This is done using a quick python script ([handscan.py](https://github.com/Freija/birdometer-ml/blob/master/code/handscan.py)). The script loops over all images (```jpg``` format) inside a directory. It also loads the CSV file that contains the labels that are already available. If an image has not been labeled yet, it displays the image using ```matplotlib.pyplot``` and asks on the command line if there is a bird or not. In addition, if there is a bird, it asks if the bird is drinking or not (beak in feeder hole). Finally it displays the answers and asks to confirm if they are correct. All answers are recorded as y/n in the CSV file behind the image name.

<p>
<img src="{{ site.baseurl }}/images/birdometer/labelling-example3.png" alt="labeling" style=" width: 100%;"/>
<em><small> A screen-shot of the labeling process. </small></em>
</p>

I labeled over 3800 images, excluding beak-only pictures (see the two example pictures in the [False triggers](#false) section) from the bird-present group. It takes about 5 seconds per image. The current labeled set has (excluding invalid entries) 664 images with a bird (class 1), 3194 images without a bird (class 0) for a total of 3858 labeled images. It is clear that the labeled data has many more class 0 images than class 1.

The labeled data is then divided into a training set (80% or 2314 images), cross validation set (10% or 772 images) and test set (10% or 772 images). The cross validation set is used to compare different classifiers and different parameters for each classifier. The test set will be used after the classifier is fine-tuned to determine the expected precision of the classifier.

<div id='Preprocessing'/>
### Image preprocessing
The images are 720x480 pixels RGB, this means that the maximum number of features is 1036800. The current pre-processing reduces this to 10368 by reducing the number of pixels but keeping the RGB information. The reduction factor is a parameter to the image processing code and can therefore easily be changed.
<p>
<img src="{{ site.baseurl }}/images/birdometer/preprocessed-example.png" alt="preprocessed" style=" width: 49.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/processed-example.png" alt="processed" style=" width: 49.5%;"/>
<em><small> Example of an original (left) and preprocessed (right) image.</small></em>
</p>
Clearly, the preprocessed image above is very coarse and the color of the bird is similar as the background in most cases. The number of pixels can easily be tweaked in the future as part of the classifier optimization. Examples of things to try would be to increase number of pixels (increasing the number of features) and/or to cut away parts of the image that have the feeder and are not expected to contain useful pixels for the classification.

<div id='Classification'/>
## Image classification

<div id='Overview'/>
### Overview
The plan is to test out different classifiers algorithms to detect if there is a bird in the image or not. Currently implemented, as a baseline, is a kNN classifier using ```KNeighborsClassifier``` from ```sklearn```. Next up are logistic regression, neural network and support vector machine.
Then, once the best classifier has been identified , I will train and use the test-set to understand the precision of the classifier. It will then be deployed on the Raspberry Pi to classify new images while in memory. The idea is to not save any image that does not have a bird. Depending on the performance, a continuous image-capture mode can be tested an implemented.

The current code can be found in the Jupyter Notebook [here](https://github.com/Freija/birdometer-ml/blob/master/code/Birdometer.ipynb).

**A closer look at the data**<br>
By doing dimensionality reduction, we can try to understand the data a bit better. Is there a clear separation between bird/non-bird pictures? If not, why not? What are the problem images and is there any other processing we can do to improve separation?

As a starter, a principal component analysis (PCA) uses a Singular Value Decomposition of the data to project it, in our case, to a two dimensional space. ```sklearn.decomposition.PCA``` is used to find the principal components and the first two principal components are plotted. The following snippet is adapted from the ```sklearn``` example. ```X_train``` and ```y_train``` are the features and labels (0 or 1) for the training set.

{% highlight python %}
from sklearn.decomposition import PCA
pca = PCA(n_components=2, svd_solver='randomized',
          whiten=True).fit(X_train)
birds_pca = pca.transform(X_train)
# birds_pca is a NumPy array that contains the two components for each image.
# [[-0.91417182 -0.58942785], ..]
# The rest is basically just for plotting
colors = ["#27AE60", "#E74C3C"]
plt.figure(figsize=(15,5))
# Set the axes limits, accounting for the marker sizes as well
plt.xlim(birds_pca[:,0].min(), birds_pca[:,0].max() + 0.05)
plt.ylim(birds_pca[:,1].min(), birds_pca[:,1].max() + 0.2)
for i in range(len(birds_pca)):
    plt.text(birds_pca[i,0], birds_pca[i,1], str(y_train[i]), color=colors[y_train[i]])
plt.xlabel("pca1")
plt.ylabel("pca2")
{% endhighlight %}
Which gives the following result:
<p>
<img src="{{ site.baseurl }}/images/birdometer/pca.png" alt="preprocessed" style=" width: 99.5%;"/>
<em><small>Plot showing the two components (pca1 and pca2) as extracted by the PCA algorithm. In green (red) are the images without (with) a hummingbird.</small></em>
</p>
From the two-dimensional plot showing the two principal components above, it is clear that there are different clusters of images and within each cluster, both class 0 (no bird, green) and class 1 (bird, red) are represented typically. We can understand this as follows:

 - The images have differences in shading and contrast depending on whether the sun is out or on the time of day.

 - Each cluster is likely a cluster of images taken in similar lighting (time of day, shade, sun).

 - Within a group of images taken in similar conditions, there are images with and without birds.

Next, it makes sense to look at a few images to investigate. Specifically, there are two interesting groups:

 * group 1: pca1 < -0.5 and pca2 > 0.5. The classes are pretty well separated in this grouping.

 * group 2: pca1 > 1.0 and pca2 < 0.0. The classes seem less well separated in this grouping.

This following plot shows group 1 in blue and group 2 in purple.
<p>
<img src="{{ site.baseurl }}/images/birdometer/pca-groups.png" alt="preprocessed" style=" width: 99.5%;"/>
<em><small>Plot showing the two components (pca1 and pca2) as extracted by the PCA algorithm. In green (red) are the images without (with) a hummingbird.</small></em>
</p>
When looking through examples of both classes in this groups, it is clear that the shade and background play an important role in how well the classes can be separated. See examples below.
<p>
<img src="{{ site.baseurl }}/images/birdometer/group1_comparison.png" alt="group 1 comparison" style=" width: 99%;"/>
<em><small> Group 1 class 1 (left) and class 0 (right) typical examples.</small></em>
</p>

<p>
<img src="{{ site.baseurl }}/images/birdometer/group2_comparison.png" alt="group 2 comparison" style=" width: 99%;"/>
<em><small> Group 2 class 1 (left) and class 0 (right) typical examples.</small></em>
</p>
For group 1, there is a large contrast between the bird (in the shade) and the background (sunny). The distance in the (pca1, pca2) space is therefore larger than for group 2, where the contrast is minimal. A change to the hardware, assuring that the bird is always in the shade, can help in this example.

The next step is to implement different classifiers.


[This project page will be updated with more insights and results as they are being uncovered and produced.]
<div id='Deployment'/>
## Current status
### Hardware
The Birdometer itself is currently in the prototyping phase: I am trying out different ways to attach the devices to the Birdometer in a way that makes the alignments reproducible. The addition of additional sensors and cameras is under investigation.
### Raspberry Pi software
The app is deployed on the Birdometer using Docker. The code itself is simple. The idea is to extend it to only save the images that have a bird once the classifier is trained and deployed.

<div id='Future'/>
## Future ideas
### Extending the Birdometer
Extend the Birdometer with additional sensors:

  1. An acoustic sensor. Maybe wing beat speed (frequency) varies between different birds.

  2. An interrupt sensor at the feeder-hole. This can be used as a trigger for additional images that can then be used for the identification of individual birds.

  3. A load-cell. Measuring the weight versus time will give an idea of the nectar consumption over time. Combined with the interrupt sensor, we can estimate the consumption per visit or bird.

  4. Additional cameras to add different angles.


### Recognizing individual birds
One of the next things to try out is a kind of hummingbird facial/body recognition: can we distinguish between different individual birds? For this, adding the additional sensors (see above) will likely help.

### Deploying additional Birdometers
It would be interesting to deploy the Birdometer at different locations and gather the data on a single webpage. Maybe we can identify the same individual bird visiting multiple feeders?
