---
layout: archive
title: "Sitemap"
permalink: /sitemap/
author_profile: false
sitemap: false
---

{% include base_path %}

Quick links to the public pages and posts on this site. Machines can use the [XML sitemap]({{ base_path }}/sitemap.xml).

<div class="sitemap__section">
  <h2>Pages</h2>
  <ul class="sitemap__list">
    <li><a href="{{ base_path }}/">Home</a></li>
    <li><a href="{{ base_path }}/blog/">Blog</a></li>
    <li><a href="{{ base_path }}/research/">Research</a></li>
  </ul>
</div>

<div class="sitemap__section">
  <h2>Posts</h2>
  <ul class="sitemap__list">
{% for post in site.posts %}
    <li>
      <a href="{{ base_path }}{{ post.url }}">{{ post.title }}</a>
      <span class="sitemap__item-meta">{{ post.date | date: "%B %-d, %Y" }}</span>
    </li>
{% endfor %}
  </ul>
</div>
