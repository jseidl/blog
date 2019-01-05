---
title: VirtuaPlant
author: Jan Seidl
type: page
date: 2015-03-30T14:12:28+00:00
---
## Introduction

VirtuaPlant is a Industrial Control Systems simulator which adds a &#8220;_similar_ to real-world control logic&#8221; to the basic &#8220;read/write tags&#8221; feature of most PLC simulators. Paired with a game library and 2d physics engine, VirtuaPlant is able to present a <acronym title="Graphical User Interface">GUI</acronym> simulating the &#8220;world view&#8221; behind the control system allowing the user to have a vision of the would-be actions behind the control systems.

## Motivation

I regularly receive emails asking for PLC simulators suggestions and I really didn&#8217;t find many options rather than simple soft-PLCs with no embedded control logic (mostly because this is implemented according to client&#8217;s needs) thus leaving to the user the job of implementing their own control logic behind.

Also, many demos are made with custom-built or off-the-shelf kits involving hardware which raises the cost of the simulators for newcomers and low budget users and the idea of having everything (Control System + Plant parts) on software came to my mind.

Finally, _the &#8220;Gamification&#8221; paradigm is hype, cool and makes the learning process even more phun!_

## Inspiration

The inspiration for starting with a bottle filling factory came from <a href="http://electrical-engineering-portal.com/plc-implementation-of-the-bottle-filling-application" title="PLC implementation of the bottle-filling application" target="_blank">an article</a> found on electrical-engineering-portal.com named &#8220;PLC implementation of the bottle-filling application&#8221;.

Even the project being inspired on the process described in the article, the process implemented in the Bottle-filling factory isn&#8217;t 100% faithful, but hey, it works!

## Overview

All the software is written in (guess what?) Python. The idea is for VirtuaPlant to be a collection of different plant types using different protocols in order to be a learning platform and testbed.

The first release introduces a as-simple-as-it-can-get one-process &#8220;bottle-filling factory&#8221; running Modbus as its protocol.

### World View

World View consits on the game and 2d physics engine, simulating the effects of the control systems&#8217; action on virtual (_cyberz!_) assets.

It uses python&#8217;s [pygame][1] and [pymunk][2] ([Chipmunk][3] engine for python &#8212; intended to be replaced by pybox2d due the lack of swept collision handling which currently limits us a little).

<img src="https://wroot.org/wp/wp-content/uploads/2015/03/worldview.png" alt="VirtuaPlant - World View" width="602" height="386" class="aligncenter size-full wp-image-52044" srcset="https://wroot.org/wp/wp-content/uploads/2015/03/worldview.png 602w, https://wroot.org/wp/wp-content/uploads/2015/03/worldview-300x192.png 300w" sizes="(max-width: 602px) 100vw, 602px" />

### PLC controller

The soft-plc is implemented over the [pymodbus][4] library which runs on a separate thread in the World View component and shares its context (i.e. Registers/Inputs/Tags) with the World View functions in order to simulate assets being &#8220;plugged in&#8221; to the controller.

### HMI

The HMI is written using GTK3 and is quite dead simple. Also runs pymodbus client on a separate thread and connects over TCP/<acronym title="Internet Protocol">IP</acronym> to the server (so it could be technically on a separate machine), constantly polling (i.e. reading) the server&#8217;s (soft PLC in World View) tags. Control is also possible by writing in the soft-PLC tags.

<img src="https://wroot.org/wp/wp-content/uploads/2015/03/hmi.png" alt="Poor Man&#039;s HMI" width="331" height="369" class="aligncenter size-full wp-image-52040" srcset="https://wroot.org/wp/wp-content/uploads/2015/03/hmi.png 331w, https://wroot.org/wp/wp-content/uploads/2015/03/hmi-269x300.png 269w" sizes="(max-width: 331px) 100vw, 331px" />

### Attack scripts

You didn&#8217;t thought I was leaving this behind, did you? The phun on having a World View is to see the results when you start messing around with the soft-PLCs tags! Some pre-built scripts for determined actions are available so you can _unleash the script-kiddie on yourself_ and make the plant go nuts! **YAY!**

<img src="https://wroot.org/wp/wp-content/uploads/2015/03/spill.png" alt="Attacking VirtuaPlant" width="602" height="386" class="aligncenter size-full wp-image-52046" srcset="https://wroot.org/wp/wp-content/uploads/2015/03/spill.png 602w, https://wroot.org/wp/wp-content/uploads/2015/03/spill-300x192.png 300w" sizes="(max-width: 602px) 100vw, 602px" />

## Demo

Check out this YouTube video demonstrating the attacks on the VirtuaPlant Bottle-Filling Factory:

{{< youtube kAfV8acCwfw >}}

## Download

All the code can be found under the [VirtuaPlant GitHub repository][5].

## Installation requirements

The following packages are required:

  * PyGame
  * PyMunk
  * PyModbus (requires pycrypto, pyasn1)
  * PyGObject / GTK

On Debian-based systems (like Ubuntu) you can apt-get the packages which are not provided over pip:

<pre class="lang:sh decode:true " >apt-get install python-pygame python-gobject python-pip</pre>

Then install the pip ones:

<pre class="lang:sh decode:true " >pip install pymunk
pip install pymodbus
pip install pyasn1
pip install pycrypto
</pre>

or install all of the pip packages by using our provided requirement.txt file:

<pre class="lang:sh decode:true">pip install &lt; requirements.txt</pre>

## Running

Enter the `/plants` directory, select the plant you want (currently only one available) and start both the world simulator and the HMI with the `start.sh` script. Parts can be ran individually by running `world.py` and `hmi.py` (self-explanatory). All the attack scripts are under the `/attacks` subdirectory.

## Future

### The following plant scenarios are being considered:

  * Oil Refinery Boiler
  * Nuclear Power Plant Reactor
  * Steel Plant Furnace

### The following protocols are being considered:

  * DNP3 (based on [OpenDNP3][6])
  * S7

Suggestions? Found bugs? Wanna help? Hit me an email or send me a message on Twitter! Contact information on [my about page][7].

 [1]: http://pygame.org/ "PyGame"
 [2]: http://pymunk.readthedocs.org/en/latest/ "PyMunk Documentation"
 [3]: http://chipmunk-physics.net/ "Chipmunk Physics"
 [4]: http://pymodbus.readthedocs.org "PyModbus Documentation"
 [5]: http://github.com/jseidl/virtuaplant "VirtuaPlant @ GitHub"
 [6]: http://www.automatak.com/opendnp3/ "OpenDNP3"
 [7]: https://wroot.org/about/ "About"
