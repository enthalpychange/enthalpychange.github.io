---
layout: page
title: Tags
permalink: /tags/
---

{% comment %}
site.tags contains all the tags
for each tag in site.tags:
  tag[0] is the tag name
  tag[1] is a list of pages with that tag
pages have properties like title and url

https://jekyllrb.com/docs/variables/
{% endcomment %}


{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
<h4 id="{{ tag[0] }}"><b>{{ tag[0] }}</b></h4>
<ul>
{% for page in tag[1] %}
<li><a href="{{ page.url }}">{{ page.title }}</a></li>
{% endfor %}
</ul>
{% endfor %}



