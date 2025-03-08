---
layout: page
title: Tags
permalink: /tags/
---

{% for tag in site.tags %}
  <h2 id="{{ tag[0] }}">{{ tag[0] | capitalize }}</h2>
  <ul>
    {% for post in tag[1] %}
      <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
        <small>{{ post.date | date: "%B %-d, %Y" }}</small>
      </li>
    {% endfor %}
  </ul>
{% endfor %}