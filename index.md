---
layout: blag

---

# blog dot vox dolo dot me

{% for post in site.posts limit:2 %}
  <div id='latest'>
    <h2><a href='{{ post.url }}'>{{ post.title }}</a></h2>
    <div id='when'>{{ post.date | date:"%B %d, %Y" }}</div>
    <div id='what'>
      {{ post.excerpt }}
    </div>
    <p id='actions'>
      <a href='{{ post.url }}#disqus_thread'>comments</a>
      &brvbar;
      <a href='{{ post.url }}'>read more&hellip;</a>
    </p>
  </div>
{% endfor %}

## Articles

{% for post in site.posts %}
* [{{ post.title }}]({{ post.url }}) &brvbar; {{ post.date | date:"%B %d, %Y" }}
{% endfor %}
