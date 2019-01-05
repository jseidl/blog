---
title: Terminal auto-lock with zsh and vlock
author: Jan Seidl
type: post
date: 2012-01-18T22:33:27+00:00
url: /posts/terminal-auto-lock-with-zsh-and-vlock/
categories:
  - Hardening
tags:
  - bash
  - linux
  - shell
  - terminal
  - unix
  - zsh

---
I&#8217;m always concerned about leaving terminal sessions open. I&#8217;ve used for many and many years the `$TMOUT` environment variable to close my sessions if idle for `N` seconds.

Just by exporting the `TMOUT` variable to the number of desired timeout seconds will close your shell (Bash, Ksh, Zsh and some others).

The following example will timeout in 300 seconds (5 minutes)

{{< highlight bash >}}
export TMOUT=300
{{< /highlight >}}

I am currently reading the book [Secure Coding: Principles & Practices][1] and the authors cited this timeout technique as pretty ineffective since it annoys more than it helps. I was obliged to agree. I got pretty mad with it some good times.

So I started looking for alternatives.

I&#8217;ve found a console application called `<a href="http://freecode.com/projects/vlock">vlock</a>`. It should be available on most distro&#8217;s repositories.

Just invoke `vlock` and the terminal session will be locked awaiting the user password to unlock. Pretty nice. Locking is definitely better than killing the session.

So I just started to wonder how to integrate `vlock` with `zsh` and after some research I&#8217;ve discovered that the shell will only be killed within `TMOUT` if no [trap function][2] for signal `ALARM` is set. 

If you set an `ALARM` trap function, it will be called instead of killing the session. Perfect.

So I ended up with this in my `.zshrc`:

{{< highlight bash >}}
export TMOUT=600
function TRAPALRM() { vlock }
{{< /highlight >}}

And now `zsh` locks my sessions after 10 minutes. It&#8217;s working perfectly even within `tmux`.

@UPDATE
  
As the comment from the reader **Ehtesh Choudhury** we can accomplish that in `tmux` only by adding to your configuration:

{{< highlight bash >}}
set -g lock-command vlock
{{< /highlight >}}

 [1]: http://www.securecoding.org/
 [2]: http://zsh.sourceforge.net/Doc/Release/Functions.html#Trap-Functions
