---
layout:   post
title:    "Add a user to a group from the OS X command line"
date:     2013-07-26 12:00:00
category: quiver
tags:     osx cli tip
icon:     terminal
---

Similar to the usermod command under Linux, you can add a user to a group from the command line in OS X too.

## Linux example:

{% highlight bash %}
sudo usermod -a -G GROUPNAME USERNAME
# eg:
sudo usermod -a -G wheel karl
{% endhighlight %}

## OS X Example:

{% highlight bash %}
sudo dseditgroup -o edit -a USERNAME -t user GROUPNAME
# eg:
sudo dseditgroup -o edit -a karl -t user wheel
{% endhighlight %}

Source: <http://superuser.com/questions/214004/how-to-add-user-to-a-group-from-mac-os-x-command-line>
