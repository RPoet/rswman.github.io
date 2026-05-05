---
layout: default
title: Articles
description: "Articles"
---

# Articles

<ul class="post-list">
{% assign article_posts = site.posts | where_exp: "post", "post.tags contains 'article'" %}
{% for post in article_posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
  </li>
{% endfor %}
</ul>
