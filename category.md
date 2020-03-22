---
title: Category
layout: default
---

<style>
    h1 {
        display: inline;
    }
    ul {
        list-style-type: none;
    }
</style>

<div class="post-list">
<!-- categories.html -->
{% assign items = site.categories %}
{% assign default = "uncategorized" %}

{% for item in items %}
{% unless item[0] == default %}
<h1><a href="#{{ item[0]| downcase }}">{{ item[0] }} ({{ item[1]| size }})</a></h1>&nbsp;&nbsp;
{% endunless %}
{% endfor %}

{% for item in items %}
{% if item[0] == default %}
<h1><a class="post-title" href="#{{ item[0]| downcase }}">{{ item[0] }} ({{ item[1]| size }})</a></h1>&nbsp;&nbsp;
{% endif %}
{% endfor %}


<hr/>

<ul>
    {% for item in items %}
    {% unless item[0] == default %}
    <li>
        <h1 id="{{ item[0]| downcase }}">{{ item[0] }} ({{ item[1]| size}})</h1>
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
        <h1 id="{{ item[0]| downcase }}">{{ item[0] }} ({{ item[1]| size}})</h1>
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