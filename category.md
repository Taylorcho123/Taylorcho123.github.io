---
title: Category
layout: default
---
<div class="post-list">
<!-- categories.html -->
{% assign items = site.categories %}
{% assign default = "uncategorized" %}

{% for item in items %}
{% unless item[0] == default %}
<h1><a href="#{{ item[0]| downcase }}">{{ item[0] }} ({{ item[1]| size }})</a></h1>
{% endunless %}
{% endfor %}

{% for item in items %}
{% if item[0] == default %}
<a class="post-title" href="#{{ item[0]| downcase }}">{{ item[0] }} ({{ item[1]| size }})</a>
{% endif %}
{% endfor %}


* * *

***

*****

- - -

---------------------------------------

<ul>
    {% for item in items %}
    {% unless item[0] == default %}
    <li>
        <h3 id="{{ item[0]| downcase }}">{{ item[0] }} ({{ item[1]| size}})</h3>
        <ul>  
            {% for post in item[1] %}
            <li><a href="{{ post.url }}">{{ post.title }}</a> <time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%d %b %Y" }}</time></li>
            {% endfor %}
        </ul>
    </li>
    {% endunless %}
    {% endfor %}

    {% for item in items %}
    {% if item[0] == default %}
    <li>
        <h3 id="{{ item[0]| downcase }}">{{ item[0] }} ({{ item[1]| size}})</h3>
        <ul>  
            {% for post in item[1] %}
            <li><a href="{{ post.url }}">{{ post.title }}</a> <time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%d %b %Y" }}</time></li>
            {% endfor %}
        </ul>
    </li>
    {% endif %}
    {% endfor %}
</ul>
{% assign items = nil %}
{% assign default = nil %}
</div>