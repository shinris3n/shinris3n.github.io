---
layout: default
title:  "News"
author: shinris3n
---

<h5><a href="{{site.url}}"> &lt;&lt; home </a></h5>
<p></p>

# News
{% for tag in site.tags %}
  {% assign t = tag | first %}
  {% assign posts = tag | last %}

<ul>
{% for post in posts %}
  {% if post.tags contains 'news' %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date" style="font-size:0.75em;">({{ post.date | date: "%b %-d, %Y" }})</span>
  </li>
  {% endif %}
{% endfor %}
</ul>
{% endfor %}
