---
layout: page
title: Tags
permalink: tags
---

{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
  <details>
    <summary><strong>{{ tag[0] }}</strong> {{ tag[1].size }} post(s)</summary>
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
