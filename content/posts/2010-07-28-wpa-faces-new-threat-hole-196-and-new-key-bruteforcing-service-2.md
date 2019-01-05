---
title: WPA faces new threat ‘Hole 196’ and new key bruteforcing service
author: Jan Seidl
type: post
date: 2010-07-28T16:30:14+00:00
url: /posts/wpa-faces-new-threat-hole-196-and-new-key-bruteforcing-service-2/
categories:
  - Network Security
tags:
  - bruteforce
  - cracking
  - mitm
  - wifi
  - wpa

---
WPA [was][1] [hit][2] [hard][3] these days by a rumor of a new threat called &#8216;Hole 196&#8217;. 

The hole would allow GTK (Group Temporal Key) spoofing forcing users to send the rogue AP their private key information leading to a awesome no-footprinted (medium is over air!) [man-in-the-middle][4] attacks.

To understand transient keys over WPA, Md Sohail Ahmad from AirTight said:

> WPA2 uses two types of keys: 1) Pairwise Transient Key (PTK), which is unique to each client, for protecting unicast traffic; and 2) Group Temporal Key (GTK) to protect broadcast data sent to multiple clients in a network. PTKs can detect address spoofing and data forgery. &#8220;GTKs do not have this property,&#8221; according to page 196 of the IEEE 802.11 standard. (&#8230;) Because a client has the GTK protocol for receiving broadcast traffic, the user of that client device could exploit GTK to create its own broadcast packet. From there, clients will respond to the sending MAC address with their own private key information. 

The solution would be some kind of GTK signing so it could be validated though the endpoint (holding the PTK) protecting itself from this spoof attack.

[The discussion][5] now is around if it is really a flaw/threat or is just another shenanigan from [AirTight][6] to gain attention from venture fundings, but it seems (this time) to be a legitimate issue.

I&#8217;ve read on [Bruce][7] that a new service called &#8220;[WPA Cracker][8]&#8220;. The service consists into [brute-forcing][9] your WPA keys with (a rich and huge database of) dictionary attacks, in many languages on their cracking-cluster of several machines.

Back in 2008, [Chad Perrin has discussed][10] that cloud computing could be used to distributed cracking / bruteforcing. Some projects have managed that like [Free Rainbow Tables][11] ([DistRTGen][12] @ Boinc).

The Church of WiFi maintains a huge database of WPA Rainbow Tables but the caveat is that keys are salted with the network&#8217;s <acronym title="Service set identifiier">SSID</acronym>.

> Yes, the Church Of Wifi has put a large rainbow table collection online. However, there are a few ways in which this collection has not met our needs. The first is that since each handshake is salted with the ESSID of the network, you have to build a unique set of rainbow tables for each network that you’d potentially like to audit. The Church Of Wifi has gone to heroic efforts to build tables for the 1000 most popular ESSIDs, but we find that this is often not enough. If someone has enabled WPA encryption on their wireless network, chances are that they’ve changed their ESSID to something that’s not very common as well.
> 
> Additionally, since they had to build so many sets, they had to limit the size of their dictionary in order to keep the resulting tables manageable. We feel that 1,000,000 words is really not large enough to do a comprehensive search, and that the way the dictionary was constructed discounts some of the specifics for WPA network password requirements. WPA Cracker provides a service that can crack the PSK of a network with any ESSID, using a dictionary that is several orders of magnitude larger.

_From [WPA Cracker&#8217;s <acronym title="Frequently Asked Questions">FAQ</acronym> on &#8220;Aren&#8217;t there Rainbow Tables?&#8221;][13]_

That&#8217;s a good approach since keys tend to have a fixed size (or better, users tend to make them the least necessary to attend the requisites) so you don&#8217;t spend computational power trying password that wouldn&#8217;t fit them.

WPA Cracker may not be a threat at all, since even with a huge cluster it&#8217;s limited do dictionary attacks. If you use a dictionary password, you _deserve_ to be cracked.

 [1]: http://wifinetnews.com/archives/2010/07/researchers_hints_8021x_wpa2_flaw.html "Researcher Gives Clues about WPA2 Flaw"
 [2]: http://www.networkworld.com/newsletters/wireless/2010/072610wireless1.html "WPA2 vulnerability found "
 [3]: http://www.darknet.org.uk/2010/07/wpa2-vulnerability-discovered-hole-196-a-flaw-in-gtk-group-temporal-key/ "WPA2 Vulnerability Discovered – “Hole 196″ – A Flaw In GTK (Group Temporal Key)"
 [4]: /tag/mitm
 [5]: http://wifinetnews.com/archives/2010/07/researchers_hints_8021x_wpa2_flaw.html#comments
 [6]: http://www.airtightnetworks.com/WPA2-Hole196
 [7]: http://www.schneier.com/blog/archives/2010/07/wpa_cracking_in.html
 [8]: http://www.wpacracker.com/index.html
 [9]: /tag/bruteforce
 [10]: http://blogs.techrepublic.com.com/security/?p=702
 [11]: http://www.freerainbowtables.com/
 [12]: http://boinc.freerainbowtables.com/distrrtgen/
 [13]: http://www.wpacracker.com/faq.html#3