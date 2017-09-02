---
layout: default
title: Welcome
---
# freija.net

Welcome. I am an experimental physicist interested in experimental design, data, science and other things. My current goal is to explore fun hardware projects and data science, in particular machine learning.

# Current projects
{% for project in site.projects reversed%}
<h2>{{ project.title }}</h2>
<ul class="tags">
{% for tag in project.tags %}
  <li><a href="/tags#{{ tag }}" class="tag">{{ tag }}</a></li>
{% endfor %}
</ul>
<img src="{{ site.baseurl }}/images/{{ project.primary_image }}" style=" width: 100%;"/>
{{ project.long_description}}
<br>
The code is on Github: <a href="{{ project.github_link }}">{{project.github_link}}</a>.
{% break %}
{% endfor %}


# Last blog post
{% for post in site.posts limit: 5%}
  <li><span>{{ post.date | date_to_string }}</span> » <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
    {% for tag in post.tags %}
      <a href="/tags#{{ tag }}" class="tag">{{ tag }}</a>
    {% endfor %}
{% endfor %}
