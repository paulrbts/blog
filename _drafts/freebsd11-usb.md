---
layout: post
title: Installing FreeBSD on a USB stick for Mac
meta: Sometimes it's undesirable to install an OS to your primary system disk, you might prefer to to put it on a USB stick. This guide walks through the process of installing FreeBSD 11.0 to a USB stick on a Mac.
category: os
tags: [FreeBSD, installation, mac]
---
<h3 class="page.title">
  {% if page.title %}
    <a href="{{ site.baseurl }}{{ page.url }}">{{ page.title }}</a>
  {% endif %}
</h3>

**{{ page.date | date_to_long_string }}**

___
The disk needs to be formatted with a GUID Partition Table (GPT), otherwise the Mac won't recognised it as a bootable volume at boot time.
