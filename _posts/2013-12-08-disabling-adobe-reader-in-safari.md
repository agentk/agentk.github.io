---
layout:   post
title:    "Disabling Adobe Reader for PDF’s in Safari 7 – OS X Mavericks"
date:     2013-12-08 12:00:00
category: quiver
tags:     osx mavericks safari adobe pdf tip
icon:     apple
---

Don’t like the Safari 7 PDF support using Adobe Reader? Why not change it to use Preview.

(Requires terminal and administrator password)

1. Close Safari and open a terminal
2. Move the Safari plugins for Adobe Reader some where safe:
    {% highlight bash %}
    mkdir -p ~/Backup/Internet\ Plug-Ins
sudo mv /Library/Internet\ Plug-Ins/AdobePDF* ~/Backup/Internet\ Plug-Ins/
{% endhighlight %}
3. Make sure Safari is not set to ignore PDF’s:
    {% highlight bash %}
defaults write com.apple.Safari WebKitOmitPDFSupport -bool NO
{% endhighlight %}
4. Launch Safari again – Click a PDF link – Enjoy

