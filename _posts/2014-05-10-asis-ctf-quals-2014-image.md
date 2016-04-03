---
layout: post
title: ASIS CTF Quals 2014 - Image
---
This is a very easy challenge. The first thing we see when we look at a hex dump
of the file is the file signature:

{% highlight text %}
4E 45 53 1A
{% endhighlight %}

Which is the plaintext ```NES``` followed by a MS-DOS end-of-file. This is the
ROM file for a game to run on Nintendo NES.

Fire up your favorite emulator, and play through the first level.

In the second level, the layout of the level spells out the words "FLAG = 8 BIT
RULES".

Since challenge didn't accept this a a flag, I had to message an admin via IRC
to discover that the flag is actually ```8bit_rulez```
