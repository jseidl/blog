---
title: Fixing ‘guake can not init’ and buggy tab titles on Backtrack 5 R3
author: Jan Seidl
type: post
date: 2012-12-04T11:29:09+00:00
url: /posts/fixing-guake-can-not-init-and-buggy-tab-titles-on-backtrack-5-r3/
categories:
  - Systems Administration
tags:
  - backtrack
  - guake
  - linux
  - terminal

---
Hi Folks, I&#8217;m a heavy [Guake Terminal][1] user and I just installed the latest Backtrack 5 revision (R3) and installed Guake on it. I was surprised when I tried to start by getting the message:

> &#8220;Guake can not init!\n\nGconf Error.\nHave you installed guake.schemas properly?&#8221;

Long story short, it seems that the package bundled with this version of Backtrack (and maybe others) have a bug into the debian package file/script that installs the `guake.schemas` file into an erroneous location (`/usr/etc/gconf/schemas/`).

The location should be `/etc/gconf/schemas/guake.schemas`. You&#8217;ll have to `mkdir` this `schemas` directory under `/etc/gconf` and `mv` or `ln -s` this file there. Problem solved.

This solution was found [here][2].

## Buggy tab titles

Other thing that annoys me a LOT is that some versions (0.4.1 and maybe older ones) of Guake have a bug in the tab titles setting mechanism. It simply goes blank and you&#8217;re left with a blank title when you try to change it.

You&#8217;ll have to hack up a Guake file to fix that:

  1. Edit guake.py (on BT5R3 is at /usr/lib/guake/guake.py
  2. Comment out the following code on line 983: `dialog.destroy()`
  3. Copy that same code (uncommented of course) below the `if` statement (about line 987)
  4. Save and restart Guake
  5. Enjoy!

This solution was found [here][3].

Well, that&#8217;s all for now! Hope that this hint helps someone!

 [1]: http://guake.org/
 [2]: http://guake.org/ticket/52
 [3]: http://askubuntu.com/questions/18975/can-i-change-the-name-of-the-guake-tab-to-show-the-current-command