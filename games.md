---
layout: home
show_excerpts: true
title: "Category: Games"
permalink: /category/games/
pagination:
  category: games
  enabled: true
  per_page: 10
  permalink: /page/:num/
  sort_field: 'date'
  sort_reverse: false
---

{% if page.pagination.category %}
<h1>Category archives: {{ page.pagination.category | capitalize }}</h1>
{% else %}
<h1>Archives - {{ page.title }}</h1>
{% endif %}xxx
