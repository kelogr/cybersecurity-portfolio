---
layout: page
title: Academic Reports
icon: fas fa-graduation-cap
order: 2
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