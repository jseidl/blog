---
title: Introducing VirtuaPlant 0.1
author: Jan Seidl
type: post
date: 2015-03-30T14:13:12+00:00
url: /posts/introducing-virtuaplant-0-1/
categories:
  - SCADA/ICS
tags:
  - chipmunk
  - gtk
  - ics
  - modbus
  - plc
  - pymunk
  - python
  - scada
  - simulator
  - software

---
Today I&#8217;m releasing a project I&#8217;ve been working on for the past week which I called VirtuaPlant (yeah I now, I suck at giving names).

_From the [project page][1]:_

> VirtuaPlant is a Industrial Control Systems simulator which adds a “similar to real-world control logic” to the basic “read/write tags” feature of most PLC simulators. Paired with a game library and 2d physics engine, VirtuaPlant is able to present a <acronym title="Graphical User Interface">GUI</acronym> simulating the “world view” behind the control system allowing the user to have a vision of the would-be actions behind the control systems.

This first version features a dead simple single process of filling bottles with small pellets (It was supposed to be liquid but I still haven&#8217;t figured out how to deal with liquids on the engine) but the _fun is guaranteed_!

## Motivation

The idea came after receiving the billionth email asking for recommendations for a PLC simulator, since I dislike the idea of simply recommending one of the soft PLCs with no logic inside which simply allows the user to read and write stuff, I&#8217;ve decided to write something more which contained some logic and also allowed the user to see the effects of their actions. 

Then while doing my daily readings I&#8217;ve stumbled upon <a href="http://electrical-engineering-portal.com/plc-implementation-of-the-bottle-filling-application" title="PLC implementation of the bottle-filling application" target="_blank">this technical article</a> and thought that it could easily be reproducible over software. Enter VirtuaPlant!

## Just shut up, show me the goodies!

The main screen is the &#8220;game screen&#8221; or &#8220;world view&#8221; which simulates the real world being controlled by the PLC. This uses the physics engine to render the actions and also features a modbus server &#8220;connected&#8221; to it.

<img src="https://wroot.org/wp/wp-content/uploads/2015/03/worldview.png" alt="Fill, fill, fill!" width="602" height="386" class="aligncenter size-full wp-image-52044" srcset="https://wroot.org/wp/wp-content/uploads/2015/03/worldview.png 602w, https://wroot.org/wp/wp-content/uploads/2015/03/worldview-300x192.png 300w" sizes="(max-width: 602px) 100vw, 602px" />

I&#8217;ve also made a poor-man&#8217;s HMI to get the field-feel, give feedback on the TAGs status and provide minimum control (Start/Stop). This has a modbus client connected over TCP the modbus server (PLC) on the World View.

<img src="https://wroot.org/wp/wp-content/uploads/2015/03/hmi.png" alt="Still less ugly than most HMIs I&#039;ve seen on field" width="331" height="369" class="aligncenter size-full wp-image-52040" srcset="https://wroot.org/wp/wp-content/uploads/2015/03/hmi.png 331w, https://wroot.org/wp/wp-content/uploads/2015/03/hmi-269x300.png 269w" sizes="(max-width: 331px) 100vw, 331px" />

And the cherry on top, some attack scripts to make everything a mess! Make the nozzle open all the time flooding everything, or the motor on all the time making all the bottles empty or stopping the whole process.

<img src="https://wroot.org/wp/wp-content/uploads/2015/03/spill.png" alt="Spill all the things!" width="602" height="386" class="aligncenter size-full wp-image-52046" srcset="https://wroot.org/wp/wp-content/uploads/2015/03/spill.png 602w, https://wroot.org/wp/wp-content/uploads/2015/03/spill-300x192.png 300w" sizes="(max-width: 602px) 100vw, 602px" />

## Demo

Here&#8217;s a video of VirtuaPlant in action:

{{< youtube kAfV8acCwfw >}}

## I wanna get it! Gimme!

Rush to the [project page][1] to find out more about the project, get the download links and instructions.

## Help wanted

If you know pybox2d or are a talented designer, your help is needed in order to make VirtuaPlant even better!

The following improvements are to be made:

  * Better UI for World View and HMI
  * Move from Chipmunk to Box2d
  * Create more plant scenarios

Contact me if you feel like you could help on these matters!

 [1]: /projects/virtuaplant