---
layout: default
---
  asdfasdf
  {% assign category = page.category | default: page.title %}
  {% for post in site.categories[category] %}
  	* 11
  {% endfor %}