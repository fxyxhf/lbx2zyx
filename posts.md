---
layout: archive
title: "Blog"
permalink: /posts/
author_profile: true
---

{% for post in site.posts %}
<h3><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h3>
<span style="color:#666;">{{ post.date | date: "%Y-%m-%d" }}</span>
<p>{{ post.excerpt }}</p>
<hr>
{% endfor %}
