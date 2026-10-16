---
layout: page
title: Mi estantería
permalink: /estanteria/
---

Los libros que he reseñado hasta ahora, del más reciente al más antiguo:

<ul class="shelf">
{% for post in site.categories.resenas %}
  <li class="book">
    <div>
      <div class="book-title"><a href="{{ post.url | relative_url }}">{{ post.libro | default: post.title }}</a></div>
      <div class="book-author">{{ post.autor }}</div>
    </div>
    <span class="rating">{{ post.valoracion }}</span>
  </li>
{% endfor %}
</ul>
