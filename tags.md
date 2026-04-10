---
layout: default
title: Tags
permalink: /tags/
---

<h1>Tags</h1>

<div class="tags-index">
  {% assign tags = site.tags | sort %}
  {% for tag in tags %}
    <a href="#{{ tag[0] }}" class="tag">{{ tag[0] }} <span class="count">({{ tag[1].size }})</span></a>
  {% endfor %}
</div>

{% for tag in tags %}
  <section id="{{ tag[0] }}" class="tag-section">
    <h2>{{ tag[0] }}</h2>
    <ul class="post-list">
      {% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
      {% for post in tag[1] %}
        <li>
          <span class="post-meta">{{ post.date | date: date_format }}</span>
          <h3>
            <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
          </h3>
          {% if post.tags.size > 0 %}
            <div class="post-tags-mini">
              {% for t in post.tags %}<span class="mini-tag">#{{ t }}</span>{% endfor %}
            </div>
          {% endif %}
        </li>
      {% endfor %}
    </ul>
  </section>
{% endfor %}