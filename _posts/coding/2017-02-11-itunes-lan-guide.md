---
layout: post
title: Shared iTunes Library on LAN Tutorial
meta: Sharing an iTunes library across the network for multiple audio output options. Use a single iTunes library for multiple devices, and listen to music on different areas of the network.
category: coding
tags: [macOS, audio]
draft: true
---
<h3 class="page.title">
  {% if page.title %}
      <a href="{{ site.baseurl }}{{ page.url }}">{{ page.title }}</a>
  {% endif %}
</h3>

**{{ page.date | date_to_long_string }}**

___
<h4>Scenario:</h4>
* your entire music library is managed through iTunes
* you have an iPhone, which syncs music from your iTunes library
* you can listen to music from this computer, but not from devices other than your iPhone (and any other iDevices syncing to that machine)
* you have a stereo in the living room, one in the bedroom and another in the garage

If you're familiar with this scenario and you like to listen to music a lot, or at least be free to access your entire library without having to move it with you all the time, then you might also be frustrated that the only easy way to do it is to put it all on your iDevice (which means buying one large enough) or take out a subscription to a streaming service.

I'm not a big fan of streaming services. I like to have things available in perpetuity. They have their advantages but I don't see them as a solution, rather, they're *part* of a solution. But that isn't really what this article's about. It's about removing the need for that just so you can share your own music -- with yourself.

iTunes is often lambasted for its inflexibility and closed attitude to user input. One thing is pretty sure though, it's the easiest and best interface for syncing music and other data between your computer and your iPhone, so we're going to stick with it.

<h5>What are we doing?</h5>
I store my central music library in full resolution, usually ALAC, on a Mac running as a server. It has speakers hooked up to it. I want to be able to pick up my iPhone, open *Remote* and play music on that machine. Additionally, I can control the destination of that machine because, of course, it has AirPlay built-in, so the full-res audio is always available to send to other machines.

Now that's standard, you can do that anyway. But as this machine is a server, I don't want to *sync* my iPhone with this machine. I want my daily driver (and any other machines on the network I grant access to) to be able to access that music and be able to sync it with the iPhone. I want to manage my iTunes library, manage my iPhone and store iPhone backups on my daily driver. To me, this feel like the most correct way of using the server as a central server because generic data is stored centrally but personal elements are on my local machine.

All of that's pretty straightforward using other guides on the internet: look for tutorials on 'how to store music on an external drive', but they don't cater for keeping the iTunes library file (iTunes Library.itl) available to both the server and any local machines. This is an issue because it causes conflicts if two instances of iTunes try to access it at the same time. The simple solution is to keep iTunes running on the server, but then you can't access it on another machine. Keep iTunes closed on the server until you need it on that machine? That's a pain if it's running headless and anyway, you need to start access it, open iTunes, then start choosing what to play.

The aim of this guide is to ensure you have your iTunes library stored on a central machine, serving it up to other computers for direct library management and outputting audio to AirPlay devices. Well, actually that part's easy, but with a little scripting you can make it as simple as opening an application and you bypass the errors caused by shared library files.

As a bonus tip, you can use [Airfoil and Airfoil Satellite](https://www.rogueamoeba.com/airfoil/) to make this machine an AirPlay client so it can *receive* audio. Why would you want that? It's about remote functionality: you want to stream a radio station, so you pick up your iPhone, open the station's app and output it to that machine. I find that simpler than sitting at the machine, firing up a browser, heading to the website and selecting the station. If your machine is headless then that option isn't there anyway, so this is extra helpful.

There are some elements of this that won't be covered, such as setting the sleep/wake cycle (use `pmset`) or Wake on LAN.

___

<h4>Getting things ready</h4>

The possible output interfaces for this are:
* any/every computer running iTunes
* AirPlay devices
* synced iDevices

This guide assumes you're using a Mac as a server to host your music library (it could be an old Mac or a desktop that's sits there, plugged into the network, in which case it's useful to put it to use when it's not otherwise being used). Actually, wherever your library resides it functions as a server. For the sake of continuity and clarity, we'll call this **mac-server**, while **owner** refers to the user on the **mac-server** with ownership of the music library. You'll need to replace **mac-server** with the address of your central machine (*e.g.* mac-mini.local) and replace **owner** with the user shortname of the owner. Your machine needs to be awake whenever you try and access it, so either keep it always, have a sensible sleep/wake schedule or use Wake On LAN.

<h5>Remote machine</h5>
Put yourself behind this machine and prep it as follows:

1. Enable *File Sharing* (just the ~/Music folder for security):
    * System Preferences > Sharing > File Sharing
    * *Only these users* > [**owner**] > *Read & Write*
    * Note the access address, below the green light: usually the computer name preceded by afp:// or smb:// depending on which protocol you're using
2. Enable *Remote Apple Events*:
    * System Preferences > Sharing > Remote Apple Events
    * *Only these users* > [**owner**]
3. Grab the IP address:
    * System Preferences > Network > Advanced > TCP/IP
4. Ensure iTunes is a login item on **mac-server** and therefore always running
    * System Preferences > Users & Groups > **owner** > Login Items
5. Quit iTunes

We need to quit iTunes at the end here so we can set up other machines using the same iTunes Library.itl file.

<h5>Daily drivers</h5>
What happens when I open iTunes on my workstation, or any other machine that doesn't host the music library? Firstly, it looks to the local .itl file and local media. This won't tally with the media being on a remote machine. We need to tell the local instance of iTunes to look to the remote .itl file *and* media library. The problem here, hinted at above, is that only one instance at a time can use that file. You can't have iTunes open on the **mac-server** and then open it on another machine, pointing to the same iTunes Library.itl -- this will throw you an error.

On the local machine(s):

1. Quit iTunes
2. Mount the remote volume, if it isn't mounted already
2. Hold *Alt* while opening iTunes
3. Select the remote iTunes Library.itl:
  * /Volumes/**mac-server**/Users/**owner**/Music/iTunes/iTunes Library.itl

This should automatically locate the media but, if it can't, then:

1. Go to iTunes' Preferences and locate the media library:
  * iTunes Preferences > Advanced > iTunes Media folder location
  * /Volumes/**mac-server**/Users/**owner**/Music/iTunes/iTunes Media

___
<h4>Making it easy</h4>

1. Open *Applescript Editor*
2. Insert the following code, editing it to match the IP address and file sharing  of **mac-server** that you gathered earlier:

```applescript
tell application "iTunes" of machine "eppc://[IP.ADDRESS]"

	quit
end tell

delay 2

tell application "Finder"
	try
		mount volume "[afp://mac-server.local/owner]"
	end try
end tell

tell application "iTunes"
	activate
end tell
```
Now just compile it to check for errors by clicking the hammer. It will ask for the password of the **owner** on the **mac-server**. Save the script somewhere on you local machine somewhere logical, as you want to save it and forget it. Call it something like "Open-iTunes.scpt".

Here's what you just did:

* Quit iTunes on the remote machine
* Waited 2 seconds (this is optional)
* Mounted the media library of the remote machine
* Opened iTunes locally

Now when we have a script that will cleanly open iTunes for us, we should write one to do the opposite. That way, when we've finished using iTunes locally it'll leave our **mac-server** ready for the next time we use it.

<h5>Reversing the thread</h5>
1. Create a new file in *Applescript Editor*, notice the syntax is subtley different this time:

```applescript
tell application "iTunes" to quit

set remMachine to "eppc://owner@[IP.ADDRESS]"

tell application "Finder" of machine remMachine
	open file "Macintosh HD:Applications:iTunes"
end tell
```

Save it as "Close-iTunes.scpt" in the same location as the *Open* script.

<h5>Single-click bliss</h5>

1. Open *Automator*
2. Go to *Actions > Utilities* and select *Run AppleScript*
3. Paste this code:

```
on run {input, parameters}

	(run script "/path/to/Open-iTunes.scpt")

	return input
end run
```

Save it as an Application called something like "Open iTunes".

Now for closing it down:

1. Create a new Automator document
2. Go to *Actions > Utilities* and select *Run AppleScript*
3. Paste this code:

```
on run {input, parameters}

	(run script "/path/to/Close-iTunes.scpt")

	return input
end run
```
Save it as an *Application* called something like "Close iTunes".

You can make this more convenient by removing the local instance of iTunes from your Dock and replacing it with these two Automator applications. It can also help to use the iTunes icon for the icons of these applications, as a quick reminder. Finally, just make sure iTunes is running on the **mac-server** so the script will run.
