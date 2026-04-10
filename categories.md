---
layout: default
title: Categories
permalink: /categories/
---

<h1>Categories</h1>

<div class="tags-index">
  {% assign categories = site.categories | sort %}
  {% for category in categories %}
    <a href="#{{ category[0] }}" class="category">{{ category[0] }} <span class="count">({{ category[1].size }})</span></a>
  {% endfor %}
</div>

{% for category in categories %}
  <section id="{{ category[0] }}" class="tag-section">
    <h2>{{ category[0] }}</h2>
    <ul class="post-list">
      {% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
      {% for post in category[1] %}
        <li>
          <span class="post-meta">{{ post.date | date: date_format }}</span>
          <h3>
            <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
          </h3>
          {% if post.categories.size > 0 %}
            <div class="post-tags-mini">
              {% for c in post.categories %}<span class="mini-category">📁 {{ c }}</span>{% endfor %}
            </div>
          {% endif %}
        </li>
      {% endfor %}
    </ul>
  </section>
{% endfor %}