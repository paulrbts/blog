---
layout: post
title: Removing duplicates from the iTunes Library
meta: iTunes may 'randomly' download or copy additional versions of the same song. These aren't covered by iTunes' built-in function to remove duplicates, so here's another way.
category: software
tags: [macOS, audio]
---
<h3 class="page.title">
  {% if page.title %}
      <a href="{{ site.baseurl }}{{ page.url }}">{{ page.title }}</a>
  {% endif %}
</h3>

**{{ page.date | date_to_long_string }}**

___
#### The lowdown
I realised recently that iTunes had downloaded or copied additional versions of some songs.
I don't know why, or when, this happened, but it meant that my folders had two versions of some songs which can take up quite a lot of storage space.
In total I had 507 unwanted items.
The reason they went unnoticed is that iTunes itself ignored them, so they were under my radar.
It wasn't until I started using `cmus` that I started noticing all these extras.
Navigating to the directory and printing a list will look something like this:

```zsh
cd ~/Music/iTunes/iTunes Media/Music/Camel/Breathless
ls -l
01 Breathless.m4a
01 Breathless 1.m4a
02 Echoes.m4a
02 Echoes 1.m4a
03 Wing and a Prayer.m4a
03 Wing and a Prayer 1.m4a
```
##### The fix
The first thing to do is navigate the `~/Music/iTunes/iTunes Media/Music` directory and generate a list of anything which might be a duplicate.
Duplicates of this nature have a '1' at the end of the file name, before the suffix, so a simple `find` would be:
```zsh
find . -type f -name "* 1.*" > remove.txt
```

This creates a file in the `~/Music/iTunes/iTunes Media/Music` folder called 'remove.txt', only putting files in there, and only files who have [space]1. in their file name.

Open the file in your favourite text editor, *e.g.* Atom, Sublime *etc.* and take a look at it.
There might be some casualties of war (files who genuinely have the number 1 in their name), so it's worth looking through the lines and removing those.
They stand out pretty easily, it doesn't take long.

You'll notice that each line is a different file, and also begins './', providing a relative path.
If you do a 'Find and Replace' to Find "./" and Replace with "/Users/[you]/Music/iTunes/iTunes Media/Music/" then you'll have a correct, absolute path to each file.

One last problem is that shell scripts don't like white spaces.
Nixcraft has an excellent guide on removing files but it doesn't work well with white spaces, so creates an extra layer of complexity.

The following command (courtesy of Stack Overflow user 'Alex L'), run in your $TERMINAL, will turn each line into a separate `rm` command and run it:
```zsh
while read line; do rm "$line"; done < remove.txt
```

##### The clean-up
You can check via Finder or command line if it's done the job, or just run the initial `find` again, but with a new filename.
This should return just the files you manually excluded earlier (the non-casualties of war).

##### Sources
* [My question at Stack Overflow](https://stackoverflow.com/questions/42675009/delete-files-from-a-list-in-a-text-file)
* [Nixcraft](https://www.cyberciti.biz/faq/deleting-bulk-files-in-unix-linux-bsd-apple-osx/)
