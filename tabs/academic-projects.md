---
layout: page
title: Academic Projects
permalink: /tabs/academic-projects/
icon: fas fa-graduation-cap
order: 1
---

En esta sección se incluyen todos mis reportes, publicaciones y proyectos académicos desarrollados durante mi formación superior y especializaciones.

## Proyectos Disponibles

| Proyecto | Fecha de Publicación |
| :--- | :--- |
{% for post in site.tags.academic %}| [{{ post.title }}]({{ post.url | relative_url }}) | {{ post.date | date: "%d/%m/%Y" }} |
{% endfor %}