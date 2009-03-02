---
title: The Rhythm is Gonna Getcha
layout: blag
---
## Posts
{% for post in site.posts %}
* {{ post.date | date_to_string }} &raquo; [{{ post.title }}]({{ post.url }})
{% endfor %}

