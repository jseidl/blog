---
title: Kentuckiana ISSA’s Metasploit Class videos available at Irongeek
author: Jan Seidl
type: post
date: 2010-05-27T00:02:42+00:00
url: /posts/kentuckiana-issas-metasploit-class-videos-available-at-irongeek/
categories:
  - Penetration Testing
tags:
  - adrian crenshaw
  - david kennedy
  - elliott cutright
  - ettercap
  - irongeek
  - martin bos
  - metasploit
  - nullthreat
  - pentest
  - purehate
  - pwncycle
  - rel1k
  - search engineering framework

---
These presentations from May 8th, 2010 performed on the Brown Hotel in Louisville, Kentucky. on exploiting with the Metasploit framework has an tremendous value. Its from this month, still fresh! Its 7 hours of presentations, it takes a while to finish (I took almost a week! phew!) but its totally worth it!

It starts with [Adrian &#8220;Irongeek&#8221; Crenshaw][1] introducing Metasploit exploiting a Windows box via `msfweb` and `msfconsole`. Pretty neat. 

Pwrcycle shows a good SYN scanning configuration on nmap with best practices for stealth using Decoys and taking advantage of fragmentation over <acronym title="Intrusion-Prevention System">IPS</acronym>/IDSs. He also describes database (sqlite) integration on MSF and importing nmap scan results (<acronym title="eXtensible Markup Language">XML</acronym>). Pwrcycle also talks a little about db_autopwn that automatically exploits the target based on their open ports (from nmap scan). Quite kiddie and stealthless (as pwncycle itself mentions on the video), but funny tough.

Another great topic handed by pwncycle is Pivoting, the technique of jumping through machines to escalate your privileges. You go from a restricted environment to a more-featured on, on the compromised machine network and account privileges.

One that is very very exciting is [Elliott &#8220;Nullthreat&#8221; Cutright][2] that introduces stack overflow on a live demo (that was really cool) of a step-by-step basic example using an outdated version of tftpd for Windows (plugged on a debugger) while spawning the `calc.exe`.

On a great overview on the Meterpreter, Metasploit&#8217;s meta-interpreter payload, [Martin &#8220;PureHate&#8221; Bos][3] shows the advanced features you can easily achieve without tons of Assembly hacking like file transfers, hash dumping, and forth. It also mentions the capability of restoring de MAC (access, more specifically) times from a file. Forensics experts, go crazy! He also talks about psexec and event log clearing. Excellent! 

This video introduces lots of concepts on post-exploitation like process migration, trace cleanup and even injecting the encoded ([shikata ga nai polymorphic encoding][4], to avoid detection from AVs) meterpreter into executables to create backdoors (the creation of a persistent meterpreter backdoor is also covered). Martin also briefly shows Metasploit Experss, that is a portable Metasploit version.

[David &#8220;ReL1K&#8221; Kennedy][5] opens his talk showing how language packs may influence the exploit since it changes memory locations and then proceeds to the opensource python-driven metasploit-integrated Social-Engineering Framework that exploits our weakest link in security: the human element.

Exploitation is done by spoofing (cloning) websites (that is excelent paired with Ettercap&#8217;s dns_spoof), spoofing Meterpreter Java Applet signature or by exploiting serious browser flaws like the [Aurora Memory Corruption][6] (used as example in video) within the Metasploit integration.

David also talks about using commom services&#8217; ports to bypass [egress filtering][7] and exploiting <acronym title="Structured Query Language">SQL</acronym> Server vulnerabilities with Metasploit and FastTrack.

Videos can be watched online at [Adrian&#8217;s website][8]. _The videos can also be downloaded in a better quality (files ~ 500MB)._

Awesome!

 [1]: http://www.irongeek.com
 [2]: http://www.nullthreat.net
 [3]: http://www.hackersforcharity.org
 [4]: http://www.offensive-security.com/metasploit-unleashed/Antivirus-Bypass
 [5]: http://www.social-engineer.org/
 [6]: http://www.metasploit.com/modules/exploit/windows/browser/ms10_002_aurora
 [7]: http://en.wikipedia.org/wiki/Egress_filtering
 [8]: http://www.irongeek.com/i.php?page=videos/metasploit-class