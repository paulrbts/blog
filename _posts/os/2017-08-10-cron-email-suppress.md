---
layout: post
title: Suppressing non-error e-mails from Cron tasks
meta: How to limit Cron's e-mails to only send errors.
category: os
tags: [admin, mac, linux, bsd]
---
<h3 class="page.title">
  {% if page.title %}
    <a href="{{ site.baseurl }}{{ page.url }}">{{ page.title }}</a>
  {% endif %}
</h3>

**{{ page.date | date_to_long_string }}**

___
##### Overview
Cron is a task executing programme which will execute tasks at defined intervals.
The process of doing so is straightforward and covered elsewhere.
Cron, by default, will email (to your prompt, next time you open one) any output from the tasks.
These can be errors from STDERR or just information, from STDOUT.
The aim of this article is to show how to suppress the non-error emails, otherwise you might get bored of receiving multiples emails a day saying telling you that your [rsync](https://rsync.samba.org/) backup or [smartmontools](https://www.smartmontools.org/) drive test completed successfully.
It would be far more useful to limit these emails to only those which report errors.

The reason for this particular post is that there seem to be some quite convoluted methods of achieving this, but I found a post over at [scottlinux.com](https://scottlinux.com/2010/12/13/cron-only-email-errors/) and [alphadevx](http://www.alphadevx.com/a/384-Suppressing-Cron-Job-Email-Notifications), which was far more succinct.

Finally, this is just for the local mail delivery (via prompt), but it will obviously work if you set up postfix or another MTA.

##### The code
Open your Crontab with

```zsh
$ crontab -e
```

and add this to the end of your task:

```zsh
>/dev/null
```

It might read:

```zsh
0 * * * * /path/to/script > /dev/null
```

That's it.

##### Sources
* [scottlinux.com](https://scottlinux.com/2010/12/13/cron-only-email-errors/)
* [alphadevx](http://www.alphadevx.com/a/384-Suppressing-Cron-Job-Email-Notifications)
