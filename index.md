---
title: The rhythm is gonna get ya!
layout: blag

---

# {{ page.title }}

{% for post in site.posts limit:1 %}
  <div id='latest'>
    <h2><a href='{{ post.url }}'>{{ post.title }}</a></h2>
    <div id='when'>{{ post.date | date:"%B %d, %Y" }}</div>
    <div id='what'>
      {{ post.content }}
    </div>
  </div>
{% endfor %}

## Articles

{% for post in site.posts %}
* [{{ post.title }}]({{ post.url }}) &brvbar; {{ post.date | date:"%B %d, %Y" }}
{% endfor %}
