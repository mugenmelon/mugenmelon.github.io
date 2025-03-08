---
layout: page
title: Tags
permalink: tags
---

{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
    {% assign name = tag[0] %}
    {% assign posts = tag[1] | sort: 'date' | reverse %}
    <details>
        <summary>
            <strong>{{ name }}</strong> - {{ posts.size }} post{% if posts.size > 1 %}s{% endif %}
        </summary>
        <ul>
            {% for post in posts %}
                <li>
                    <a href="{{ post.url }}">{{ post.title }}</a>
                    <small>{{ post.date | date: "%B %-d, %Y" }}</small>
                </li>
            {% endfor %}
        </ul>
    </details>
{% endfor %}
