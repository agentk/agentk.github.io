---
layout:      post
title:       "Remove duplicate items from the Open With menu in Finder"
description: "OS X Command line tip to clean up the Open With menu"
date:        2013-07-25 12:00:00
category:    quiver
tags:        osx cli tip
icon:        command
---

To remove duplicate items from the "Open With…"" menu in Finder:

Open a terminal and run:

{% highlight bash %}
/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/Support/lsregister -kill -r -domain local -domain system -domain user
{% endhighlight %}

Control-Option Click on Finder in the Dock and select Relaunch

Admire the clean Open With menu.

Tested on:

- OS X 10.8 Mountain Lion
- OS X 10.9 Mavericks
- OS X 10.10 Yosemite

Source: <https://discussions.apple.com/thread/4250905?start=0&tstart=0>
