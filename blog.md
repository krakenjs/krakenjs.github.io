---
title: kraken<em>js</em> news
description: Tentacle tamer training, tips and tricks
layout: blog
---

<ul>
  {% for post in site.posts %}
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    {{post.excerpt}}
  {% endfor %}
</ul>

