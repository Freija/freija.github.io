---
layout: post
title: "Testing Google Vision API - continued"
date: 2018-01-08
tags:
 - vision api
 - birdometer
 -
---
In the last post, [Testing Google Vision API]({% link _posts/2017-11-09-vision-apis.md %}), the different ways to access the Google Vision API to label photos were discussed. As examples, some of the hummingbird images from the [birdometer]({% link _projects/2017-10-21_birdometer.md %}) project were fed to the API. In this post, we will loop over all the images in the [birdometer]({% link _projects/2017-10-21_birdometer.md %}) training set and collect the labels for each one.

## The data-set

Here are two example birdometer images, each with a bird present.
<p>
<img src="{{ site.baseurl }}/images/birdometer/2017-09-20_16.25.44_1.jpg" alt="Birdometer image example" style=" width: 49.5%;"/>

<img src="{{ site.baseurl }}/images/birdometer/2017-09-20_16.25.44_4.jpg" alt="Birdometer image example" style=" width: 49.5%;"/>

<br><em><small> Birdometer images 2017-09-20_16.25.44_4.jpg (left) and 2017-09-20_16.25.44_1.jpg (right), used in the Google Vision API testing.</small></em>
</p>

These images had the following labels assigned by the Google Vision API:

  * 2017-09-20_16.25.44_4.jpg: ```red, flora, flower, leaf, plant, grass, propeller```

  * 2017-09-20_16.25.44_1.jpg: ```flora, leaf, grass, flower, plant```

During testing, it became clear that some images that contain birds, do not actually receive any bird-related labels. This is likely due to the low contrast between the green background and the green birds as well as the unusual top-view angle at which the images are recorded.

The hand-classified data-set contains (excluding invalid entries) 664 images with a bird (class 1), 3194 images without a bird (class 0) for a total of 3858 classified images.

## Results
All hand-classified images were then also labeled by the Google Vision API, using a quick Python script based on the code discussed the the last post ([Testing Google Vision API]({% link _posts/2017-11-09-vision-apis.md %})).
This script:

 1. Loops over all hand-classified images.

 2. Contacts the Google Vision API for each image (that has not been processed yet) to request annotation.

 3. Stores the labels and classification in a local CSV file.

 This resulted in 54 unique labels overall: 43 for the class 1 images (bird) and 33 for the class 0 images (no bird). There is significant overlap in labels between the two classes as we can expect from images that are so similar. The images with the birds have a larger variety of labels. This is also expected, since there is an additional element (bird) present. Using the Python ```wordcloud``` library and excluding the two most common labels: ```grass``` and ```plant```, two wordclouds are created: for the images with and without a bird. Each wordcloud shows all labels, the font size represents the frequency the label was assigned. The code can be found in the [Google Vision API Jupyter notebook](https://github.com/Freija/birdometer-ml/blob/master/code/Google_Vision_api.ipynb).

 <p>
 <img src="{{ site.baseurl }}/images/birdometer/nobird_wordcloud.png" alt="Birdometer image example" style=" width: 49.5%;"/>

 <img src="{{ site.baseurl }}/images/birdometer/bird_wordcloud.png" alt="Birdometer image example" style=" width: 49.5%;"/>

 <br><em><small> Wordclouds for the classified Birdometer images without (left) and with (right) a hummingbird present in the image.</small></em>
 </p>

The 'fauna' and 'bird' labels do not show up for the class 0 images, whereas they do for about 44.5% and 34% of the class 1 images respectively. The current method does not make use of the probabilities that the Google Vision API assigns to each label. It would be interesting to incorporate those as weights. Next up, we can use the labels of the training set to train a classifier, to be continued.

Below are all 54 unique labels with their respective frequency that they were returned for the class 0 and 1 images (percentage of images with that label). Table generated using the Google Charts API. Click on column name to order by that column. I think ```personal_protective_equipment``` might be my favorite label so far!

<html>
  <head>
    <script type="text/javascript" src="https://www.gstatic.com/charts/loader.js"></script>
    <script type="text/javascript">
      google.charts.load('current', {'packages':['table']});
      google.charts.setOnLoadCallback(drawTable);

      function drawTable() {
        var data = google.visualization.arrayToDataTable([
         ['label', 'class 0', 'class 1'],
         ['grass', 99.96869129618034, 90.51204819277109],
          ['plant', 97.99624295554165, 77.71084337349397],
          ['yellow', 91.29618033813401, 21.53614457831325],
          ['tree', 75.79837194740138, 18.22289156626506],
          ['grass_family', 71.47777082028804, 8.433734939759036],
          ['soil', 58.79774577332498, 7.680722891566265],
          ['flora', 34.18910457107076, 59.48795180722892],
          ['wheel', 33.34376956793989, 0.30120481927710846],
          ['lawn', 22.072636192861616, 1.2048192771084338],
          ['red', 8.672510958046336, 45.63253012048193],
          ['leaf', 8.077645585472762, 37.19879518072289],
          ['flower', 6.230432060112712, 33.13253012048193],
          ['tire', 4.19536631183469, 0.0],
          ['water', 3.913587977457733, 7.530120481927711],
          ['play', 3.6631183469004385, 1.5060240963855422],
          ['sunglasses', 3.1621790857858483, 0.0],
          ['car', 3.005635566687539, 0.0],
          ['eyewear', 2.34815278647464, 0.0],
          ['green', 2.34815278647464, 1.0542168674698795],
          ['automotive_tire', 1.878522229179712, 0.0],
          ['petal', 1.4715090795241077, 5.873493975903615],
          ['sunlight', 1.3462742642454602, 0.0],
          ['fun', 1.2523481527864746, 1.9578313253012047],
          ['pink', 1.1271133375078273, 3.7650602409638556],
          ['glasses', 0.9079524107701941, 0.0],
          ['wildflower', 0.5635566687539136, 2.2590361445783134],
          ['motor_vehicle', 0.21916092673763307, 0.0],
          ['organism', 0.18785222291797118, 3.7650602409638556],
          ['vertebrate', 0.18785222291797118, 16.867469879518072],
          ['magenta', 0.18785222291797118, 0.45180722891566266],
          ['vision_care', 0.09392611145898559, 0.0],
          ['personal_protective_equipment', 0.031308703819661866, 0.0],
          ['vehicle', 0.031308703819661866, 0.0],
          ['wildlife', 0.0, 0.15060240963855423],
          ['tail', 0.0, 0.15060240963855423],
          ['fauna', 0.0, 45.48192771084337],
          ['fish', 0.0, 14.006024096385541],
          ['bird_feeder', 0.0, 0.15060240963855423],
          ['beak', 0.0, 25.150602409638555],
          ['piciformes', 0.0, 0.15060240963855423],
          ['macaw', 0.0, 0.15060240963855423],
          ['lorikeet', 0.0, 0.15060240963855423],
          ['galliformes', 0.0, 0.30120481927710846],
          ['pollinator', 0.0, 1.6566265060240963],
          ['hummingbird', 0.0, 6.626506024096385],
          ['propeller', 0.0, 0.30120481927710846],
          ['marine_biology', 0.0, 0.6024096385542169],
          ['adventure', 0.0, 0.15060240963855423],
          ['screenshot', 0.0, 0.15060240963855423],
          ['chicken', 0.0, 0.30120481927710846],
          ['bird', 0.0, 33.8855421686747],
          ['gun', 0.0, 0.15060240963855423],
          ['jungle', 0.0, 0.15060240963855423],
          ['toucan', 0.0, 0.30120481927710846]
         ],
            false);
        var table = new google.visualization.Table(document.getElementById('table_div'));

        table.draw(data, {showRowNumber: true, width: '100%', height: '100%', page:'enable', pageSize:20});
      }
    </script>
  </head>
  <body>
    <div id="table_div"></div>
  </body>
</html>
