---
layout: page
title: courses
---

<h1>修課紀錄</h1>

<p>以下是大一所有修課紀錄與心得：</p>

<div class="blog-list">
  {% for post in site.posts %}
    <div class="blog-card">
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      <p class="date">{{ post.date | date: "%Y-%m-%d" }}</p>
      <p class="excerpt">{{ post.excerpt | strip_html | truncate: 100 }}</p>
      <a class="read-more" href="{{ post.url }}">閱讀更多 →</a>
    </div>
  {% endfor %}
</div>

