---
title: 'Endpoint DLP: Is hardware access control enough?'
author: Jan Seidl
type: post
date: 2011-01-26T02:30:48+00:00
url: /posts/endpoint-dlp-is-hardware-access-control-enough/
categories:
  - Data-loss prevention
  - Robotics / Automation
tags:
  - automation
  - AVR
  - DLP
  - HID
  - malware
  - microcontroller

---
I&#8217;ve stumbled on [this article][1] about using a custom-built hardware to bypass hardware enforcement on most DLP solutions available on market.

The solution uses an [Atmel&#8217;s AVR microcontroller][2] (the same on the [Arduino][3]&#8216;s I&#8217;ve been playing around lately) and the [V-<acronym title="Universal Serial Bus">USB</acronym> library][4] to create a virtual <acronym title="Universal Serial Bus">USB</acronym> device and is crafted to announce itself as [<acronym title="Human-interface device">HID</acronym>][5] (Human-interface device). What common hardware fits this description: **the keyboard**. As you are not likely to forbid keyboard access to your users (or else they wouldn&#8217;t be able to type and thus work), this will gracefully pass through many enforcements.

The <acronym title="Human-interface device">HID</acronym> protocol allows bi-directional communication and this makes a perfect vector for data transfer. 

> The <acronym title="Human-interface device">HID</acronym> protocol allows communication in both directions by sending and receiving reports and feature requests. I’ve utilised this control channel to allow the PC to transfer files over the <acronym title="Human-interface device">HID</acronym> protocol to the device (&#8230;)

Thomas Cannon (the article author) had made experiments with a <acronym title="Universal Serial Bus">USB</acronym> drive and other with an micro SD module attached to the custom-built hardware in order to store the transfered data since the AVR internal memory is quite short (maybe it&#8217;s fine for Bill Gates).

## White-listing

While I&#8217;ve already seen some DLP solutions enforcing pendrives accesses through its vendor and model, I dunno if some had already made a way to enforce a unique fingerprint for each device (maybe yes, sounds feasible).

## Can you trust drive signatures? What about the data they traffic?

What if drives signatures cannot be trusted anymore? We are left with the only enforcement left at input level: data monitoring.

Network <acronym title="Intrusion-Prevention System">IPS</acronym>/IDSs are employed to detect (and sometimes stop) rogue data running on wires. Application <acronym title="Intrusion Detection System">IDS</acronym>/IPSs are employed to detect (and sometimes stop) rogue data that come over inputs. Why not deploy <acronym title="Intrusion Detection System">IDS</acronym>/IPSs at <acronym title="Human-interface device">HID</acronym> level? They are inputs, aren&#8217;t?

Suppose you have the latest DLP solution blocking all your <acronym title="Universal Serial Bus">USB</acronym> drives, <acronym title="Compact Disc">CD</acronym> drives, Floppy drives, ZIP drives, iPhone etc etc. You also have all your network connections, email servers, <acronym title="HyperText Transfer Protocol">HTTP</acronym> traffic monitored. 

An badly intentioned user won&#8217;t be able to upload malware through the drives nor download from web or network but what would stop him from actually TYPING the payload into notepad? Every hex or worse, every bit? Yes, sounds crazy and would take lots of time but hey &#8211; never doubt a determined person. 

## Things can be automated

Ok, so you think &#8216;Hey Jan, this would never happen, you are crazy. It would be very very very hard to keep track of every 0 or 1 typed&#8217;. Sure, I completely agree. The AVR-based hardware thus, does not.

This little beauty needs a piece of code on the host machine to make the data transfer possible so it has a stage where you simply open a notepad and the microcontroller (since he is actually a keyboard) will type it for you! Wow, zero work huh?

## Seems that Moore was right after all

There are lot of microcontroller kits like BasicX, Arduino, Parallax and Pololu, just to name a few. Earlier at the São Paulo&#8217;s Hackers to Hackers Conference (H2HC) I&#8217;ve seen people using and R/C quad-copter holding an embedded system to crack WEP keys. Now we have a &#8216;robot-keyboard&#8217;. 

Hardware is getting cheaper, fast as hell. A system is as secure as the complexity need to crack it. Cheaper hardware, more processing per second, greater the complexity has to be.

I wouldn&#8217;t be surprised if the next hackers cracking machines will be actually, machines.

 [1]: http://thomascannon.net/projects/dlp-bypass/
 [2]: http://www.atmel.com/products/AVR/
 [3]: http://www.arduino.cc
 [4]: http://www.obdev.at/products/vusb/
 [5]: http://en.wikipedia.org/wiki/Human_interface_device