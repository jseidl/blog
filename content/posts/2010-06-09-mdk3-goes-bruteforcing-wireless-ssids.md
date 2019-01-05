---
title: MDK3 goes bruteforcing Wireless SSIDs
author: Jan Seidl
type: post
date: 2010-06-09T15:03:38+00:00
url: /posts/mdk3-goes-bruteforcing-wireless-ssids/
categories:
  - Penetration Testing
tags:
  - bruteforce
  - cracking
  - mdk3
  - tape
  - wifi

---
One good practice is to disable your <acronym title="Service set identifiier">SSID</acronym> broadcast so you don&#8217;t show up on the victims list. Although this doesn&#8217;t make you completely invisible, it does aid reducing ease of location. (Networks can still be located by BSSIDs).

[MDK3][1] was written by [ASPj][2] to bruteforce network SSIDs (even with wordlists).

[Tape][3] has done some testings around and described it all on [his blog post][4]. It has some videos too of the attack in progress on a 3-character-lenght <acronym title="Service set identifiier">SSID</acronym>.

MDK3 version 6 is already available with the latest release of [BackTrack 4][5] on `/pentest/wireless/mdk3`.

The Church of Wi-Fi has [some <acronym title="Service set identifiier">SSID</acronym> wordlists available at their website][6].

Good cracking!

 [1]: http://homepages.tu-darmstadt.de/~p_larbig/wlan/#mdk3
 [2]: http://homepages.tu-darmstadt.de/~p_larbig/
 [3]: http://adaywithtape.blogspot.com/
 [4]: http://adaywithtape.blogspot.com/2009/10/using-mdk3-in-backtrack-4-to-crack.html
 [5]: http://www.backtrack-linux.org/
 [6]: http://www.renderlab.net/projects/WPA-tables/