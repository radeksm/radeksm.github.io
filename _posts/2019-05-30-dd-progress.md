---
layout: post
title: How do you monitor progress of dd command?
tags:
    - linux
    - utils
author: Radosław Śmigielski
---

Why?
====
Who doesn't know the `dd` command?

And how many times have you `dd` some ISO image or other large files
and you were sitting, looking at terminal and asking questions,
how long is it going to take? how much data has been copied so far?

It was very frustrated.

And now, the old dark days are behaind, with the new `dd` option:
__`status=progress`__.

How?
====
And here is how to use it:
{% highlight bash %}
dd status=progress if=Fedora-Workstation-netinst-x86_64-29-1.2.iso of=/dev/sdb
{% endhighlight %}

Below you can see how the progress is shown:
{% highlight bash %}
$ dd status=progress if=Fedora-Workstation-netinst-x86_64-29-1.2.iso of=/dev/sdb
3202617856 bytes (3.2 GB, 3.0 GiB) copied, 7 s, 458 MB/s
{% endhighlight %}


Enjoy!
