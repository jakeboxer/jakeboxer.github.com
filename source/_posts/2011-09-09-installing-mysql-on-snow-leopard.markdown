---
layout: post
title: "Installing MySQL on Snow Leopard"
date: 2009-09-12 19:32
comments: true
categories:
---

Early on at Cisco, I realized that it's really beneficial to write down (step by step) what I've done when I install something new. It forces me to think about what I'm doing, it provides me with a guide in case I mess up and have to start over, and other people can benefit from it later.

With that in mind, I've decided to document my installation of MySQL on Snow Leopard (OS X 10.6). Hopefully, someone will get some use out of it, but if not, at least I'll have documentation of what I did.

The PATH Variable
-----------------

MySQL has a bunch of useful executables that aren't in PATH by default. I could symlink to them, but I think that's less maintainable than just appending to PATH.

1. Open the Terminal
   
1. Open up the .bash_profile file in your home directory. This is a bash script that runs every time you start the Terminal, so the PATH variable will be extended properly every time. **If you have TextMate and its UNIX command line tools installed**, do it like this:
 
   ``` bash
   mate ~/.bash_profile
   ```
   
   Otherwise, do it like this:
   
   ``` bash
   /Applications/TextEdit.app/Contents/MacOS/TextEdit ~/.bash_profile &
   ```
   
   If you use the second one, make sure to add the ampersand at the end.
1. We want to add MySQL's bin folder to PATH. MySQL's going to be installed at /usr/local/mysql, so let's add /usr/local/mysql/bin to PATH. Add the following line to .bash_profile:
   
   ``` bash
   export PATH="/usr/local/mysql/bin:/usr/local/sbin:$PATH"
   ```
 
   This replaces PATH with two new locations and the old value of path (so essentially, appending the two new locations to the beginning of PATH). You'll notice that, in addition to /usr/local/mysql/bin (which I mentioned earlier), I also added /usr/local/sbin. OS X doesn't include this in PATH by default, but I think it should, so I added it. I have no defense for that position, but this is as much a guide for myself as it is a guide for other people, so I don't need to defend it :)
   
1. Save the updated .bash_profile and close it.
   
1. Quit and reopen Terminal so that .bash_profile will be rerun (you could also run it explicitly, but I prefer quitting and reopening).
   
1. Run the following command:
   
   ``` bash
   echo $PATH
   ```
   
   You should see /usr/local/mysql/bin and /usr/local/sbin in there now.
   
1. **Edit:** Restart your computer. My installation worked without doing this, but Jon (in the comments) said he needed to do this before it worked, so you might as well.

Downloading + Installing
------------------------

I might try compiling and installing from source at some point, but I don't see any reason to when there are official OS X binaries available. Worst case scenario, it messes with a bunch of my settings, and I'll uninstall and then do it from source.

1. Go to http://dev.mysql.com/downloads/mysql/5.1.html#downloads
1. Scroll down to the Mac OS X section (near the bottom). Download the version labeled "Mac OS X 10.5 (x86_64)". **It's very important that you download the 64-bit version.** The 32-bit one will give you preference pane problems (I call them PPPs whenever I talk about them, though this is the first time I ever have). Eventually, there'll probably be a Mac OS X 10.6 version, but for now, this works perfectly.
1. Open the .dmg, then run mysql-x.x.xx-osx10.5-x86_64.pkg. Continue through all the screens without changing anything, until it finishes.
1. Optional: install the preference pane by running MySQL.prefPane.
1. Optional: make MySQL start up along with OS X by running MySQLStartupItem.pkg. I didn't do this (I often use my computer for things other than development, and I don't want to bog it down unnecessarily), so I can't provide any instruction or vouch for how well it works on Snow Leopard.

You should now be able to start MySQL by going into the MySQL preference pane and clicking "Start MySQL Server". If it doesn't work, leave me a comment, and I'll try to help you.