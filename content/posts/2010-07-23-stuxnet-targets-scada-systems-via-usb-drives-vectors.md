---
title: Stuxnet targets SCADA systems via USB drives vectors
author: Jan Seidl
type: post
date: 2010-07-23T16:19:06+00:00
url: /posts/stuxnet-targets-scada-systems-via-usb-drives-vectors/
categories:
  - Malware
tags:
  - backdoor
  - data loss
  - malware
  - rootkit
  - scada
  - siemens
  - stuxnet
  - trojan
  - wincc

---
Microsoft disclosed a [zero-day flaw on Windows Shell][1] on Friday and [Stuxnet (W32.Stuxnet)][2] is already exploiting it to gain access to <acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> systems through its attack vector.

Since <acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> systems are updated mainly by CDs or pen drives, the attack vector fits as a glove. The malware targets Siemens&#8217; Simatic WinCC software and intends to steal information like projects schematics and upload them to an external website.

From [CNet][2]:

> Once the malware locates the data it is looking for it encodes it and attempts to upload it to a remote server. The malware waits for a response from the server, which may contain more commands, he said.

Along with the data steal, Stuxnet also provides a trojan backdoor aiming Siemens services and a rootkit (to hide it from the system).

> Once the machine is infected, a Trojan looks to see if the computer it lands on is running Siemens&#8217; Simatic WinCC software. The malware then automatically uses a default password that is hard-coded into the software to access the control system&#8217;s Microsoft <acronym title="Structured Query Language">SQL</acronym> database. The password has been available on the Internet for several years, according to [Wired&#8217;s Threat Level blog][3].

Sophos has also released a video on YouTube showing a [<acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> system compromised by Stuxnet][4].



The spreading is done by using stolen/spoofed signed digital certificates:

> The malware includes a rootkit, which is software designed to hide the fact that a computer has been compromised, and other software that sneaks onto computers by using a digital certificates signed two Taiwanese chip manufacturers that are based in the same industrial complex in Taiwan&#8211;RealTek and JMicron, according to Chester Wisniewski, senior security advisor at Sophos. (Sophos has posted a video showing how a computer is infected on YouTube.) It is unclear how the digital signatures were acquired by the attacker, but experts believe they were stolen and that the companies were not involved.

Adding to this scenario, <acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> admins are not able to change the default password because it would break up software apart.

<acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> systems rely on the fact of being unplugged from networks and as [Schneier said][5], &#8220;would YOU like to be the guy that breaks all installed systems controlling valves and such by adding security that nobody demands?&#8221;.

It seems that <acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> will demand closer attention from now on since I agree with other professionals that doubts that this is the first <acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> malware.

 [1]: http://www.scmagazineus.com/flaw-uses-usb-devices-as-vector-to-steal-data/article/174873/
 [2]: http://news.cnet.com/8301-27080_3-20011159-245.html
 [3]: http://www.wired.com/threatlevel/2010/07/siemens-scada/
 [4]: http://www.youtube.com/sophoslabs#p/u/1/1UxN7WJFTVg
 [5]: http://www.schneier.com/blog/archives/2010/07/internet_worm_t.html