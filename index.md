---
layout: default
title: Home
description: "Graphics Rendering Engineer"
---

# rswman

<p class="lead">Graphics Rendering Engineer</p>

## Latest

<ul class="post-list">
{% assign article_posts = site.posts | where_exp: "post", "post.tags contains 'article'" %}
{% for post in article_posts limit:5 %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
  </li>
{% endfor %}
</ul>

## Browse

<ul class="link-list">
  <li>
    <a href="{{ '/articles.html' | relative_url }}">Articles</a>
  </li>
  <li>
    <a href="{{ '/stats.html' | relative_url }}">Stats</a>
  </li>
  <li>
    <a href="{{ '/work.html' | relative_url }}">Work Board</a>
  </li>
</ul>
