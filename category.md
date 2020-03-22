---
layout: default
---
  asdfasdf
  {% assign category = post.category | default: post.title %}
  {% for post in site.categories[category] %}
  		* 11
  	    * {{ site.baseurl }}{{ post.url }}
          {{ post.title }}
          {{ post.date | date: "%b %-d, %Y" }}
  {% endfor %}