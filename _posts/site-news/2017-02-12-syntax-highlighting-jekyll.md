---
layout: post
title: Syntax highlighting in a Jekyll blog
meta: Jekyll and Github Pages support syntax highlighting 'out of the box', but it doesn't work unless you've set it up properly.
category: site-news
tags: [jekyll, github]
---
<h3 class="page.title">
  {% if page.title %}
      <a href="{{ site.baseurl }}{{ page.url }}">{{ page.title }}</a>
  {% endif %}
</h3>

**{{ page.date | date_to_long_string }}**

___
The official documentation for Jekyll says that it:

> *has built-in support for syntax highlighting of code snippets using either Pygments or Rouge, and including a code snippet in any post is easy.
Just use the dedicated Liquid tag as follows:*

![Jekyll's output]({{ site.baseurl }}/images/jekyllSyntaxScrot.png)

Now, if you're new to Jekyll and you follow these instructions, it might not work. Here's why:

Jekyll recognises that there's code there but doesn't know what colours to use.
You need a [stylesheet detailing the css](https://gist.githubusercontent.com/demisx/025698a7b5e314a7a4b5/raw/d2086c7f59105db4da1ed8d1df8d8586666f66ea/syntax.css) for that.
Save it to your css directory as syntax.css

Because you're referring to a new stylesheet, you need to call this in your *_layouts/default.html* head, as follows:

``` html
<link href="/css/syntax.css" rel="stylesheet">
```

Lastly, just make sure your `_config.yml` is correct:
```
markdown: kramdown
highlighter: rouge
kramdown:
  input: GFM
  hard_wrap: false
  syntax_highlighter: rouge
extensions: fenced_code_blocks
```

Thanks to [@demisx for the post](https://demisx.github.io/jekyll/2014/01/13/improve-code-highlighting-in-jekyll.html) on this.
