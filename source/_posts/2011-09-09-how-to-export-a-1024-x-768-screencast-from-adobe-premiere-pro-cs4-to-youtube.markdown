---
layout: post
title: "How to export a 1024 X 768 screencast from Adobe Premiere Pro CS4 to Youtube"
date: 2009-03-22 17:40
comments: true
categories:
---

Over spring break, I made some screencasts for the [website](http://www.umoch.org/) I maintain for my job. We used a free, open-source screen recorder called [CamStudio](http://camstudio.org/). Our plan was to upload them to YouTube and embed the videos into our site from there.  The one problem: the Export Media dialog on Adobe Premiere Pro CS4 (the software I used to edit the screencasts) only has a setting for low-definition (320x240) videos. My screencasts were 1024x768, and shrinking them to 320x240 would make the video part pretty much useless.

So, I set about trying to find a setting that would work for a 1024x768 screencast. The majority of settings produced some weird-quality results, even with the quality settings turned up as high as possible, which makes me think it had something to do with a faulty interlacing/deinterlacing setting somewhere.  When I changed the one interlacing setting I was able to find, the exported videos would work fine in QuickTime, but look completely messed up in [VLC](http://www.videolan.org/vlc/) and, more importantly, YouTube.

Exporting to F4V actually did work in every media player I tried it in, but for some reason, YouTube was unable to convert it to a playable format.

Finally, after about 9 hours of fiddling with Adobe Premiere Pro's Export Media dialog, I found a setting of acceptable quality that worked on YouTube.  I doubt many other people will need this, but since I was unable to find any information about the problem I was having online, maybe I'll save another person 9 hours of work.  Here are the settings I used:

Export Settings
---------------
Format: QuickTime

Filters
-------
No Changes

Video
-----
* Video Codec: H.264
* Quality: 100
* Width: 1,024
* Height: 768
* (Width and Height are unlinked)
* Frame Rate: 30
* Field Type: Lower First *(this was the setting that deals with interlacing I believe)*
* Aspect: Square Pixels (1.0)
* Render at Maximum Depth: Checked
* Set Key Frame Distance: Unchecked
* Optimize Stills: Checked
* Frame Reordering: Unchecked
* Set Bitrate: Unchecked

Audio
-----
No Changes

Others
------
No Changes

I'm sure there are some optimizations to be made in these settings; things that are causing me to produce unnecessarily large files, things that make rendering slower, or things that, if I tweaked, would give me even better quality. However, after 9 hours, I don't care; this works, so for now, I'm done.  Hope this helps someone.