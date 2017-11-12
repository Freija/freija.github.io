---
layout: post
title: "Testing Google Vision API"
date: 2017-11-09
tags:
 - vision api
 - birdometer
---
For the [birdometer]({% link _projects/2017-10-21_birdometer.md %}) project, one of the critical steps is to identify whether or not a picture has a hummingbird in it or not. There are multiple Vision APIs available that we could potentially use for this task. This blog post discusses the tests done with the Google Vision API.

## Google Cloud Vision API
[Google Cloud Platform ](https://cloud.google.com/vision/)offers a Cloud Vision API for image content analysis, which detects the different objects and faces in an image. It has an integrated REST API.  For testing, an image can be uploaded through the website.
<p>
<img src="{{ site.baseurl }}/images/birdometer/VisionTryItOut-screenshot.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<em><small> Screenshot of the Google Cloud Platform Vision webpage, where an image can be uploaded for testing.</small></em>
</p>


## Testing the Google Vision Image Analysis
First, I manually uploaded a few images using the ```Try the API``` utility on the website, starting by a few pictures of hummingbirds taken with a DSLR camera.
<p>
<img src="{{ site.baseurl }}/images/birdometer/APItest1.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/APItest2.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/APItest4.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<em><small> Screenshot of the Google Cloud Platform Vision webpage, showing the results for three hummingbird pictures.</small></em>
</p>
That looks really good! All three pictures have ```Bird``` at the top of the list and ```Hummingbird``` is present for all of them as well. Impressive. Apparently, that first image really matches the hummingbird model very well. Time to try some of the images taken from the [Birdometer]({% link _projects/2017-10-21_birdometer.md %}). These are all lower resolution, top-view images of hummingbirds approaching and feeding from the feeder.
<p>
<img src="{{ site.baseurl }}/images/birdometer/testAPI10.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/testAPI12.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/testAPI11.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<em><small> Screenshot of the Google Cloud Platform Vision webpage, showing the results for three Birdometer images, two with and one without a hummingbird present.</small></em>
</p>

The Birdometer images are clearly more challenging, ```Bird``` is no longer the top result for the image with the hummingbird present. For the second image, there is no ```Bird``` nor ```Hummingbird```. ```Fauna``` is as close as it gets! However, the results for the image without the hummingbird does not contain the ```Fauna``` result. Perhaps we can use that.

I spent some time cropping the images, changing the exposure, go to gray scale and so on. The only clear improvement is seen when cropping, see the results for the cropped images below.

<p>
<img src="{{ site.baseurl }}/images/birdometer/testAPI14.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<img src="{{ site.baseurl }}/images/birdometer/testAPI13.png" alt="Google Vision API - Try it Out" style=" width: 99.5%;"/>
<em><small> Screenshot of the Google Cloud Platform Vision webpage, showing the results for the two cropped birdometer images.</small></em>
</p>

Interesting how the second image has ```Beak``` before ```Bird```. In any case, this looks promising. Next step is to check out the API and see if we can use our labeled data-set to test the Google Vision API's capability in finding the images with a bird in them! The labeled set was labeled using the full-sized (not cropped) images. Therefore, the initial test will be done with the full images.

## Using the REST API for bulk processing
Quick note that the services discussed below are not all free of charge, see pricing on the [Google Cloud Platform](https://cloud.google.com/). GCP is currently offering a free trial consisting of 12 Months and $300 free credit.

The labeled data-set contains over 3800 images, some of which have been uploaded to Google Cloud Storage. This means that we do not need to transfer the image with the Vision API request, speeding things up a bit.

### Using curl on command line
The GCP documents explain how to send requests to the Vision API using curl: [https://cloud.google.com/vision/docs/using-curl](https://cloud.google.com/vision/docs/using-curl).
For this, you will need to set up authentication, I'll be using an API key for the curl example. Details on the different ways to authenticate are here: [https://cloud.google.com/vision/docs/auth](https://cloud.google.com/vision/docs/auth). Let's try it out! The request itself needs to be placed inside a JSON file, request.json. Here is an example, using an image from the Google Cloud Storage that contains a hummingbird (replace the image link with the actual link):

{% highlight python %}
{
  "requests": [
    {
      "image": {
        "source": {
          "imageUri": "gs://link/to/2017-09-20_16.25.44_1.jpg"
        }
      },
      "features": [
        {
          "type": "LABEL_DETECTION",
          "maxResults": 10
        },
      ]
    }
  ]
}
{% endhighlight %}

This is the actual image:
<p>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-20_16.25.44_1.jpg" alt="Google Vision API - Try it Out" style=" width: 50.5%;"/>
<br><em><small> Birdometer image 2017-09-20_16.25.44_1.jpg, used in the curl command.</small></em>
</p>

The curl command is:
{% highlight bash%}
curl -v -s -H "Content-Type: application/json"   https://vision.googleapis.com/v1/images:annotate?key='<API KEY>'
--data-binary @request.json
{% endhighlight %}

Our label annotation request receives a JSON response of type AnnotateImageResponse, which contains the labelAnnotations.

{% highlight python %}
{
  "responses": [
    {
      "labelAnnotations": [
        {
          "mid": "/m/06fvc",
          "description": "red",
          "score": 0.967145
        },
        {
          "mid": "/m/03bmqb",
          "description": "flora",
          "score": 0.9178557
        },
        {
          "mid": "/m/0c9ph5",
          "description": "flower",
          "score": 0.81645584
        },
        {
          "mid": "/m/09t49",
          "description": "leaf",
          "score": 0.7736301
        },
        {
          "mid": "/m/05s2s",
          "description": "plant",
          "score": 0.7735228
        },
        {
          "mid": "/m/08t9c_",
          "description": "grass",
          "score": 0.57300675
        },
        {
          "mid": "/m/05xt4",
          "description": "propeller",
          "score": 0.5455734
        }
      ]
    }
  ]
}
{% endhighlight %}
Great. Or, wait, we got a response but there seems to be nothing bird or animal related in the labels: ```red, flora, flower, leaf, plant, grass, propeller```. Just to check, I uploaded the image through the webpage and we got the same response, as expected. Hmm, ```propeller``` is pretty interesting though, I wonder if it is because of the blurry wings. To really understand how well the Google Vision API performs on the labeled data set, we need to set up real bulk processing. Time to check out the Python client.


### Using Google Vision API Python Client
Find the docs for the available client libraries here: [https://cloud.google.com/vision/docs/reference/libraries](https://cloud.google.com/vision/docs/reference/libraries).

Start by setting up a service account to be able to authenticate following the recommendation: [https://cloud.google.com/docs/authentication/getting-started](https://cloud.google.com/docs/authentication/getting-started). This will make it really easy from inside the Python code. Save the JSON authetication file somewhere safe and set the environment variable ```GOOGLE_APPLICATION_CREDENTIALS``` to point to that file.

Then, we can follow the example from the API client documentation page linked above, here it is:

{% highlight python %}
import io
import os
# Imports the Google Cloud client library
from google.cloud import vision
from google.cloud.vision import types
def detect_labels(path):
    """Detects labels in the file."""
    client = vision.ImageAnnotatorClient()

    with io.open(path, 'rb') as image_file:
        content = image_file.read()

    image = types.Image(content=content)

    response = client.label_detection(image=image)
    labels = response.label_annotations
    print('Labels:')

    for label in labels:
        print(label.description)
{% endhighlight %}

Trying it out:

{% highlight python %}
getlabels.detect_labels('/home/freija/Projects/birdometer-ml/trainingdata/20
17-09-20_16.25.44_4.jpg')

Labels:
flora
leaf
grass
flower
plant
{% endhighlight %}
And here is that image, it does have a bird, despite the labels.
<p>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-20_16.25.44_4.jpg" alt="Google Vision API - Try it Out" style=" width: 50.5%;"/>
<br><em><small> Birdometer image 2017-09-20_16.25.44_4.jpg, used in the python test.</small></em>
</p>

Next step would be to loop over all the images in the training set and collect the labels for each one. We can then try kNN algorithm in the label space. I'll update here once those results are in.
