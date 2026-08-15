---
layout: blog
title: Blog
---

## Blog

{% for post in site.posts %}
<article class="blog-item">
  <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %-d, %Y" }}</time>
  <p>{{ post.excerpt | strip_html | truncate: 220 }}</p>
</article>
{% endfor %}
