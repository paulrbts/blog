---
layout: post
title: Shared iTunes Library on LAN Tutorial
meta: Sharing an iTunes library across the network for multiple audio output options. Use a single iTunes library for multiple devices, and listen to music on different areas of the network.
category: site-news
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

iTunes is often lambasted for its inflexibility and closed attitude to user input.
This isn't a position I'm going to argue with, although I find that a lot of the problems others seem to suffer with don't appear in my environment, for whatever reason.
This guide is going to be about how to make iTunes more usable on a network.

___

<h4>Getting things ready</h4>

The possible output interfaces for this are:
* any/every computer running iTunes
* AirPlay devices
* synced iDevices

This guide assumes you're using a Mac as a server to host your music library (it could be an old Mac or a desktop that's sits there, plugged into the network, in which case it's useful to put it to use when it's not otherwise being used).
Actually, wherever your library resides it functions as a server.
For the sake of continuity and clarity, we'll call this **mac-server**, while **owner** refers to the owner of the music library.
Put yourself behind this machine and prep it as follows:

<h5>Remote machine</h5>
1. Enable *File Sharing* (just the ~/Music folder for security):
    * System Preferences > Sharing > File Sharing
    * *Only these users* > [owner] > *Read & Write*
    * Note the access address, below the green light: usually the computer name preceded by afp:// or smb:// depending on which protocol you're using
2. Enable *Remote Apple Events*:
    * System Preferences > Sharing > Remote Apple Events
    * *Only these users* > [owner]
3. Grab the IP address:
    * System Preferences > Network > Advanced > TCP/IP
4. Ensure iTunes is a login item on **mac-server** and therefore always running
    * System Preferences > Users & Groups > [owner] > Login Items
5. Quit iTunes

This last point is where the complications would usually occur but, counter-intuitively, I've found it works best to keep iTunes running on this machine all the time (except when we're running iTunes on another machine -- we control the switch with scripting).
My **mac-server** runs headless and I want to be able to pick up my iPhone, open *Remote* and control the output from my  **mac-server** .
This is plugged into a set of speakers, providing audio in that room.
The audio on that machine is stored at full resolution, rather than the downscaled versions that I store on the iPhone itself.
Additionally, I can control the destination of that machine because, of course, it has AirPlay built-in, so the hi-res audio is always available to send to other machines.

What I don't want is to *sync* my iPhone with this machine.
I want to sync the music that's stored on this machine, yes, but it's not my daily driver, it's a server -- it holds music and other files so they're accessible *to* and *from* my daily workstation.
Additionally, I want to manage my iTunes library, manage my iPhone and store iPhone backups on/from my daily driver.
To me, this feel like the most correct way of using the **mac-server** as a central server while holding the personal elements on my machine.
Other people in the house can also access the **mac-server** without having access to my iPhone and I don't have to jump onto the **mac-server** every time I want to administer the library (meaning it can run headless if it's a Mac Mini, Power Mac or Mac Pro or in a cupboard if it's a laptop or iMac).

<h5>Local machines</h5>
So, what happens when I open iTunes on my workstation, or any other machine that doesn't host the music library?
Firstly, it looks to the local .itl file and local media.
This won't tally with the media being on a remote machine.
We need to tell the local instance of iTunes to look to the remote .itl file *and* media library.
The problem here, hinted at above, is that only one instance at a time can use that file.
You can't have iTunes open on the **mac-server** and then open it on another machine, pointing to the same .itl -- this will throw you an error.

So, on the local machine(s):

1. Quit iTunes locally
2. Mount the remote volume, if it isn't already
2. Hold *Alt* while opening iTunes
3. Select the remote .itl:
  * /Volumes/mac-server/Users/owner/Music/iTunes/iTunes Library.itl

This should automatically locate the media but, if it can't, then:

1. Go to iTunes' Preferences and locate the media library:
  * iTunes Preferences > Advanced > iTunes Media folder location
  * /Volumes/mac-server/Users/owner/Music/iTunes/iTunes Media

<h4>Making it easy</h4>

1. Open *Applescript Editor*
2. Insert the following code, editing it to match the IP address of **mac-server**:

```
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

3. Compile it to check for errors
    * Click the hammer

It will ask for the password of the **owner** on the **mac-server**.
Save the script somewhere on you local machine somewhere logical, as you want to save it and forget it.
Call it something like "Open-iTunes.scpt".

Here's what you just did:

1. Quit iTunes on the remote machine
2. Waited 2 seconds (this is optional)
3. Mounted the media library of the remote machine
4. Opened iTunes locally

Now when we have a script that will cleanly open iTunes for us, we should write one to do the opposite.
That way, when we've finished using iTunes locally it'll leave our **mac-server** ready for the next time we use it.

<h5>Reversing the thread</h5>
Notice the syntax is subtley different in this script:

```
    tell application "iTunes" to quit

    set remMachine to "eppc://owner@[IP.ADDRESS]"

    tell application "Finder" of machine remMachine
    	open file "Macintosh HD:Applications:iTunes"
    end tell
```

Save it as "Close-iTunes.scpt" in the same location as the *Open* script.

1. Open *Automator*
2. Go to *Actions > Utilities* and select *Run AppleScript*
3. Paste this code:

```
    on run {input, parameters}

    	(run script "/path/to/Open-iTunes.scpt")

    	return input
    end run
```

4. Save as an Application called something like "Open iTunes"

Now for closing it down:

1. Create a new Automator document
2. Go to *Actions > Utilities* and select *Run AppleScript*
3. Paste this code:
    on run {input, parameters}

    	(run script "/path/to/Close-iTunes.scpt")

    	return input
    end run
4. Save as an *Application* called something like "Close iTunes"

You can make this more convenient by removing the local instance of iTunes from your Dock and replacing it with these two Automator applications.
Finally, just make sure iTunes is running on the **mac-server** so the script will execute properly.
