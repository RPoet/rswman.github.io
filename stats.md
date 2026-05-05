---
layout: default
title: Stats
description: "rswman site page views and visitor counters."
sitemap: false
noindex: true
---

# Stats

<div class="stats-grid">
  <div class="stat-card">
    <div class="stat-label">Total views</div>
    <div class="stat-value" data-counter="site-views">-</div>
  </div>
  <div class="stat-card">
    <div class="stat-label">Total visitors</div>
    <div class="stat-value" data-counter="site-visitors">-</div>
  </div>
  <div class="stat-card">
    <div class="stat-label">Today views</div>
    <div class="stat-value" data-counter-today="views">-</div>
  </div>
  <div class="stat-card">
    <div class="stat-label">Today visitors</div>
    <div class="stat-value" data-counter-today="visitors">-</div>
  </div>
</div>

## Pages

<ul class="post-list" id="page-stats">
  <li data-path="{{ '/' | relative_url }}">
    <a href="{{ '/' | relative_url }}">Home</a>
    <span class="post-date">-</span>
  </li>
  <li data-path="{{ '/articles.html' | relative_url }}">
    <a href="{{ '/articles.html' | relative_url }}">Articles</a>
    <span class="post-date">-</span>
  </li>
  {% assign article_posts = site.posts | where_exp: "post", "post.tags contains 'article'" %}
  {% for post in article_posts %}
  <li data-path="{{ post.url | relative_url }}">
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="post-date">-</span>
  </li>
  {% endfor %}
</ul>

<script>
  (function () {
    var namespace = "rpoet-rswman-github-io";
    var api = "https://api.counterapi.dev/v1/" + namespace + "/";
    var today = new Date().toISOString().slice(0, 10);

    function pageKey(path) {
      var normalized = path.replace(/^\/rswman\.github\.io\/?/, "/");
      var key = "page-" + (normalized || "/")
        .toLowerCase()
        .replace(/[^a-z0-9]+/g, "-")
        .replace(/^-|-$/g, "")
        .slice(0, 90);
      return key === "page-" ? "page-home" : key;
    }

    function read(name, node) {
      fetch(api + encodeURIComponent(name) + "/", { cache: "no-store" })
        .then(function (response) { return response.ok ? response.json() : null; })
        .then(function (data) { node.textContent = data && typeof data.count === "number" ? data.count : "0"; })
        .catch(function () { node.textContent = "?"; });
    }

    document.querySelectorAll("[data-counter]").forEach(function (node) {
      read(node.getAttribute("data-counter"), node);
    });

    document.querySelectorAll("[data-counter-today]").forEach(function (node) {
      read(node.getAttribute("data-counter-today") + "-" + today, node);
    });

    document.querySelectorAll("#page-stats li").forEach(function (item) {
      var node = item.querySelector(".post-date");
      read(pageKey(item.getAttribute("data-path")), node);
    });
  })();
</script>
