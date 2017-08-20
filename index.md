---
layout: default
title: Welcome
---
# freija.net

Welcome. I am an experimental physicist interested in experimental design, data, science and other things. My current goal is to explore fun hardware projects and data science, in particular machine learning.

# Current projects
## whereispatrick.at
<img src="{{ site.baseurl }}/images/whereispatrick.png " alt="Drawing" style=" width: 100%;"/>

I built a website to track my father's progress as he is biking through the Andes from Ecuador to the very bottom of Argentina. He has a satellite phone that he uses to send a text message with his GPS coordinates, typically once a day. The website is private and used by family members to follow the adventure.

Whereispatrick.at displays the GPS coordinates on a Google Map. It also shows the locations of any geo-tagged photos that my father uploads to Google Drive. Thumbnails of the photos are shown when clicking on the photo markers. Tools used are:
 - Flask API and website.
 - Twilio to forward the Iridium GPS coordinate text messages to my Flask API.
 - Google Maps API.
 - Google Drive API to automatically download any new photos.
 - sklearn DBSCAN clustering algorithm to cluster photos that are taken close together.

The code is on Github: [https://github.com/Freija/whereispatrick](https://github.com/Freija/whereispatrick).

# Last blog post
{% for post in site.posts %}
  <li><span>{{ post.date | date_to_string }}</span> » <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  {{ post.excerpt }} <a href="{{ post.url }}"> Read the full post</a></li>
  {% break; %}
{% endfor %}
