---
layout: default
title: Home
permalink: /
---

# alex1701c.github.io

Welcome to my blog! Here’s a list of all posts:
{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%B %d, %Y" }}
{% endfor %}

<a class="btn btn-rss" href="/feed.xml" target="_blank">RSS</a>
