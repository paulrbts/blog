---
layout: post
title: First blog post
meta: The first blog post of my site
category: site-news
---
<h3 class="page.title">
  {% if page.title %}
      <a href="{{ site.baseurl }}{{ page.url }}">{{ page.title }}</a>
  {% endif %}
</h3>

**{{ page.date | date_to_long_string }}**

___
Here's my first blog post.
It's just a test post.

Nothing more to see here...
