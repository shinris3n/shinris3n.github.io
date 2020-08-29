---
layout: default
title:  "Resources"
author: shinris3n
---

<h5><a href="{{site.url}}"> &lt;&lt; home </a></h5>
<p></p>

# Resources
{% for post in site.posts %}
  	{% if post.category contains 'resources' %}
  		{% assign resrc_list_str = resrc_list_str | append: post.title | append: " ,,, " %}
  	{% endif %}
{% endfor %}

{% assign resrc_list = resrc_list_str | split: " ,,, " %}

{% assign resrc_list = resrc_list | uniq | sort_natural %} 

<ul>
{% for resource in resrc_list %}
{% for post in site.posts %}
  {% if post.title == resource %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
  <br>
  {% endif %}
{% endfor %}
{% endfor %}
</ul>