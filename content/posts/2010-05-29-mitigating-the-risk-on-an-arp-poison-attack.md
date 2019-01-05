---
title: Mitigating the risk on an ARP poison attack
author: Jan Seidl
type: post
date: 2010-05-29T23:44:57+00:00
url: /posts/mitigating-the-risk-on-an-arp-poison-attack/
categories:
  - Network Security
tags:
  - arp poison
  - ids
  - ipsec
  - mitm
  - network
  - ssl

---
ARP poisoning is a technique quite simple to be applied and allows traffic to be sniffer over a switched network. It can be used to sniff the connection on-the-fly and capture plain-text password or hashes. ARP poison also allows combination with other attacks such as <acronym title="Domain Name Server">DNS</acronym> spoof and packet filters in order to deploy client side exploits transparently.

This attack can only be performed from the local network because ARP packets aren&#8217;t routed so you can&#8217;t hop between LANs but it can be performed from any machine on the same network so it is a serious concern when dealing with unhappy employees, interns and industrial espionage. It is even a greater concern if you have public (or WEP encrypted) WIFI access in the same network (terrible mistake).

## Countermeasures

ARP can&#8217;t really be blocked because they resolve <acronym title="Internet Protocol">IP</acronym> addresses to MAC addresses. ARP packets must flow thru the network (unless you have ALL your MAC addresses statically configured) so machine can talk to each other. ARP doesn&#8217;t verify the requests and responses so any machine is able to respond to an ARP request in behalf of another machine making all traffic pass thru him. This is the ARP poison attack and is one of the most popular <acronym title="Man-in-the-middle">MITM</acronym> techninques.

### Subnetting

A well designed network is effective against several attacks, including and ARP poison attack. Since ARP packets can&#8217;t be routed the attack is retained at the local network so you may consider break your network into smaller subnets (VLANs). Note that attack still can be escalated if a machine that communicates to other lans is compromised.

### Encryption

Encrypted traffic can be sniffed but can&#8217;t be understood. IPSec is feasible too but you can secure your services under <acronym title="Secure Sockets Layer">SSL</acronym> so you don&#8217;t need to encrypt all your network traffic that could slow down your network.

### Static ARPs

You may write your ARP tables statically so it doesn&#8217;t need to be updated via requests and responses. This can be tough to manage so you might consider issuing this to login scripts or other configuration management software. Its like maintaining an `hosts` file.

### Inline IDSs and ARP watchers

Using an inline <acronym title="Intrusion Detection System">IDS</acronym> within your routers may early detect ARP storms and isolate the attacker. Some software may be put to watch ARP tables for changes. One ARP watch solution under UNIX systems is `<a href="http://www.arpalert.org/">arpalert</a>`.