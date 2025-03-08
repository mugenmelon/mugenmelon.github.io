---
layout: page
title: Tags
permalink: /tags/
---

{% for tag in site.tags %}
  <details>
    <summary><strong>{{ tag[0] }}</strong> ({{ tag[1].size }} posts)</summary>
    <ul>
      {% for post in tag[1] %}
        <li>
          <a href="{{ post.url }}">{{ post.title }}</a>
          <small>{{ post.date | date: "%B %-d, %Y" }}</small>
        </li>
      {% endfor %}
    </ul>
  </details>
{% endfor %}
