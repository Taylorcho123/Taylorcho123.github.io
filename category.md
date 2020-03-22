---
layout: default
---
  asdfasdf
  {% assign category = post.category | default: post.title %}
  {% for list in site.categories[category] %}
  	* 11
  {% endfor %}