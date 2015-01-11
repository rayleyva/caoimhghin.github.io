---
layout: default
title: Blog
permalink: /blog/
group: nav
weight: 4
---
<div class="home">

{% for post in site.posts %}
    <h4><a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a> <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span></h4>
{% endfor %}

</div>