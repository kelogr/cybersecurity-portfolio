---
layout: page
title: Academic Projects
permalink: /categories/
icon: fas fa-graduation-cap
order: 1
---

Aquí encontrarás mis reportes y publicaciones académicas.

## Reportes Disponibles
<ul>
  {% for post in site.tags.academic %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> — {{ post.date | date: "%B %d, %Y" }}
    </li>
  {% endfor %}
</ul>