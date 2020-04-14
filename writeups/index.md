---
layout: default
title:  "Writeups"
author: shinris3n
---
<h5><a href="{{site.url}}"> &lt;&lt; home </a></h5>
<p></p>

# Writeups
{% for post in site.posts %}
  	{% if post.category contains 'writeups' %}
  		{% assign chal_src_list = chal_src_list | concat: post.challenge-source%}
  	{% endif %}
{% endfor %}

{% assign chal_src_list = chal_src_list | uniq | sort_natural %} 

{% for chal_src in chal_src_list %}
	{{chal_src}}
	{% for post in site.posts %}
		{% if post.challenge-source contains chal_src %}

<ul>
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date" style="font-size:0.75em;">({{ post.date | date: "%b %-d, %Y" }})</span><br>
    {% comment %}
    <span class="tags" style="font-size:0.75em; font-style:italic;">Challenge Source: {{post.challenge-source}} </span><br>
    <span class="tags" style="font-size:0.75em; font-style:italic;">Challenge Category: {{post.challenge-category}} </span><br>
    <span class="tags" style="font-size:0.75em; font-style:italic;">Tags: {{post.tags | join: ", "}} </span>
    {% endcomment %}
  </li>
</ul>

		{% endif %}
	{% endfor %}
{% endfor %}