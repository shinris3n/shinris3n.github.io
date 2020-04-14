---
layout: default
title:  "News"
author: shinris3n
---

<h5><a href="{{site.url}}"> &lt;&lt; home </a></h5>
<p></p>

# News

<ul>
{% for post in site.posts %}
  {% if post.categories contains 'news' %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date" style="font-size:0.75em;">({{ post.date | date: "%b %-d, %Y" }})</span>
  </li>
  {% endif %}
{% endfor %}
</ul>