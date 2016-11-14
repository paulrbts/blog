---
layout: default
title: First blog post
meta: The first blog post of my site
category: site-news
---
<h3 class="post.title">
  {% if post.title %}
      <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
  {% endif %}
</h3>

Here's my first blog post. It's just a test post.

Nothing more to see here...
