---
title: The rhythm is gonna get ya!
layout: blag

---

# {{ page.title }}

{% for post in site.posts limit:1 %}
  <div id='latest'>
    <h2><a href='{{ post.url }}'>{{ post.title }}</a></h2>
    <div class='when'>{{ post.date | date:"%B %d, %Y" }}</div>
    {{ post.content }}
  </div>
{% endfor %}

## Posts

{% for post in site.posts %}
* [{{ post.title }}]({{ post.url }}) &brvbar; {{ post.date | date:"%B %d, %Y" }}
{% endfor %}
