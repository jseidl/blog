---
title: Command-line sudo and chmod under Windows
author: Jan Seidl
type: post
date: 2010-05-21T04:51:04+00:00
url: /posts/command-line-sudo-and-chmod-under-windows/
categories:
  - Access Control
tags:
  - cacls
  - chmod
  - permissions
  - runas
  - sudo
  - windows

---
I&#8217;m quite not a fan of <acronym title="Graphical User Interface">GUI</acronym> utilities but I&#8217;m a great fan of command-line and batch processing.

I&#8217;ve recently changed profiles under a Windows box and had to move some files over them. At first wondered about logging in as the old user, copy all the files to a temp folder, log back as my new user and copy them to my new folder. But hell, too much time spent.

There&#8217;s must be a command-line solution, even on Windows!

So, I started Googling around for `sudo` like commands and then I&#8217;ve found Windows&#8217; built-in `runas`

## runas

Basic syntax for `runas` is as follow:

{{< highlight bash >}}
runas /user:MACHINENAME\username command
{{< /highlight >}}

Example: Spawning an Administrator shell

{{< highlight bash >}}
runas /user:DEVEL\Administrator cmd
{{< /highlight >}}

See all the options on `runas /?`.

## cacls

This is the Windows&#8217; `chmod`. It allows you to change files and dirs permissions for Windows&#8217; ACL&#8217;s under Read/Write/Change/Full Control schema. 

CACLS (Change ACLs) also lets you to edit the ACL instead of replacing it through the `/E` directive. It has three basic operations: **Grant `/G`**, **Replace `/P`** or **Demote `/D`**.

{{< highlight bash >}}
cacls FILENAME /P USER:PERMS
{{< /highlight >}}

Access type (PERMS) can be:

N
:   None (only with /P)

R
:   Read

W
:   Write

C
:   Change

F
:   Full Access

Example: Granting user full-access under certain directory

{{< highlight bash >}}
cacls mydir\* /P jseidl:F
{{< /highlight >}}

Of course, you must have the propper permissions to set the permissions, so it&#8217;s a good companion for `runas`.

Hope it helped!
