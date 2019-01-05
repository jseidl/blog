---
title: Downgrade HTTPS connections to HTTP using Ettercap filters
author: Jan Seidl
type: post
date: 2010-07-13T17:42:06+00:00
url: /posts/downgrade-https-connections-to-http-using-ettercap-filters/
categories:
  - Penetration Testing
tags:
  - ettercap
  - filter
  - http
  - ssl

---
Ettercap is a great tool for <acronym title="Man-in-the-middle">MITM</acronym> poisoning and sniffing. Everyone on Infosec should have played with it (or Cain) at least once. 

## Man-In-The-Middle

<acronym title="Man-in-the-middle">MITM</acronym> attacks are pretty easy to perform on a local network but the tools tend to crash a LOT. Cain (Windows) is a little more stable than Ettercap but I prefer it over Cain because it doesn&#8217;t spoof <acronym title="Secure Sockets Layer">SSL</acronym> that I consider too loud depending on the attack. _NOTE: Ettercap runs better on text mode._

## Filters

Well, another nice feature of Ettercap are its filters. You can do lot of stuff while playing with them. The nicest toy I&#8217;ve found to play around so far is content rewriting (but I think custom packet injection can be even funnier). [Irongeek has played with Ettercap Filters in the past][1] to rewrite `img` tags.

## Sniffing while <acronym title="Man-in-the-middle">MITM</acronym>

Sniffing plaintext passwords are just easy. Either Cain and Ettercap are built to detect common strings containing passwords but <acronym title="Secure Sockets Layer">SSL</acronym> has made this kind of sniffing impossible and many sites are using it at least for the login processes. 

So while we wait for the super-quantum-computers to break 256-bit AES encryption, we may consider avoiding <acronym title="Secure Sockets Layer">SSL</acronym> for the period we&#8217;re sniffing so I&#8217;ve thought that filters could be perfect for that.

What would define where the login form data will be sent? The `form`&#8216;s `action` field. So if I can interfere in the <acronym title="HyperText Transfer Protocol">HTTP</acronym> response I can send the login data ANYWHERE.

I&#8217;ve decided to just downgrade the <acronym title="Secure Sockets Layer">SSL</acronym> because I always tend to make the least noise I can (because we don&#8217;t want to get caught by the forensics, do we?). I could redirect the request to a specially crafted site or so but it would be much more noticeable.

## What if <acronym title="Secure Sockets Layer">SSL</acronym> is required on server-side

No problem, <acronym title="Secure Sockets Layer">SSL</acronym> on server-side can be a requirement but by the time the server complains, data was already sent over in plain-text. :)

## Getting things done

You can get [my filter on my github page][2].

Just run the attack with the filter (assuming router is 192.168.0.1 and victim is 192.168.0.100): 

{{< highlight bash >}}
ettercap -T -q -F hrf.ef -M ARP:remote /192.168.0.1/ /192.168.0.100/
{{< /highlight >}}

You should see the following output:

{{< highlight text >}}
ettercap NG-0.7.3 copyright 2001-2004 ALoR & NaGA

Content filters loaded from hrf.ef...
Listening on eth0... (Ethernet)
(...)
[HTTP Response Filter] Encoding zapped.
[HTTP Response Filter] Encoding zapped.
[HTTP Response Filter] Encoding zapped.
[HTTP Response Filter] *** HTTPS ZAPPED from response
[HTTP Response Filter] Encoding zapped.
[HTTP Response Filter] *** HTTPS ZAPPED from response
[HTTP Response Filter] *** HTTPS ZAPPED from response
(...)
{{< /highlight >}}

And your victim will no longer receive (nor send) any _https_ string anymore.

## Quick note about request / response filtering

Sometimes you may have to comment one leg (request / response) out of the filtering or you will get redirection loops (like while tampering Facebook connections). Also, if the request is already under https, you won&#8217;t be able to filter it. The beauty of this attack is disallowing your victim to escape your domain to a secure zone. 

HTH!

 [1]: http://www.irongeek.com/i.php?page=security/ettercapfilter
 [2]: http://github.com/jseidl/etter.filter.hrf/
