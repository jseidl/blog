---
title: Preventing your servers from downgrade attacks
author: Jan Seidl
type: post
date: 2010-05-22T21:33:15+00:00
url: /posts/preventing-your-servers-from-downgrade-attacks/
categories:
  - Authentication
tags:
  - attack
  - downgrade
  - ntlm
  - prevention
  - ssl

---
One of the coolest attacks is forcing a downgrade between the client and server, making the server believe that client has support only for older and insecure versions of your protocol. This works with Windows&#8217; NTLM authentication and with <acronym title="Secure Sockets Layer">SSL</acronym>, mostly.

## How does &#8216;downgrade attack&#8217; work?

Downgrade attacks are born from a misconfiguration and takes place within <acronym title="Man-in-the-middle">MITM</acronym> attacks. Privilege is escalated by trapping the connection request and thus forcing the use of an older protocol version with known security issues by faking the client-side accepted protocol versions. Then the weak protocol is attacked and access is escalated.

<!--more-->

_Some softwares like Cain (for Windows) can perform both attacks (ARP Poisoning + NTLM Downgrade). Cain can also spoof the challenge in order to make the cracking easier._

## Why support old protocols?

You may have old clients in your network or software with limited functionality. What I say? Dump them. If you have to open a hole into your setup to accomodate something, you are doing it wrong! There&#8217;s almost no control about a <acronym title="Man-in-the-middle">MITM</acronym> attack due protocol&#8217;s weakness, since everything is spoofed. It&#8217;s better clarify your client of the risks he&#8217;s assuming on doing that. 

## What are the chances?

There&#8217;s no way to do remote ARP poisoning because ARP packets are not sent through routers but it can be easely done from inside. Internal fraud can be done though <acronym title="Man-in-the-middle">MITM</acronym> attacks and paired with a downgrade attack can leverage session hijacking on local servers.

## How to stop the downgrade?

Simply configure your software to accept only the latest (or at least the latest without serious flaws) protocol versions.

### OpenSSL

OpenSSL accepted protocol&#8217;s configuration is commonly done in application level. 

#### Apache&#8217;s mod_ssl

To disable SSLv2 (unsecure) under apache and keep only the good encryption types, configure your httpd.conf (or apache2.conf &#8212; depends on distro) to the following:

{{< highlight apacheconf >}}
SSLProtocol -ALL +TLSv1
SSLCipherSuite ALL:!ADH:!NULL:!EXP:!SSLv2:!LOW:!MEDIUM:RC4+RSA:+HIGH
{{< /highlight >}}

#### STunnel

Stunnel supports protocols since SSLv2 through latest TLS versions but you can ensure the protocol version with the following:

{{< highlight toml >}}
sslVersion = TLSv1 TLSv1.1 TLSv1.2
{{< /highlight >}}

#### Tomcat

I&#8217;ve heard that some newer versions of JVM doesn&#8217;t even accept SSLv2, but you can use TLSv1 <del datetime="2015-04-03T18:25:53+00:00">or SSLv3.</del> Do not use SSLv3 since it&#8217;s vulnerable to the POODLE attack.

Just add the following to your connector configuration:

{{< highlight xml >}}
               &lt;Connector port="443" ...
               scheme="https" secure="true"
               SSLEngine="on" 
               SSLProtocol="TLSv1"
               ..." />
{{< /highlight >}}

_There are more configuration required for <acronym title="Secure Sockets Layer">SSL</acronym>&#8217;ing on Tomcat. For this, please visit [this reference][1]._

_<del datetime="2015-04-03T18:22:57+00:00">I&#8217;ve used SSLv3 on examples because it&#8217;s the version in widest use but TLSv1 is also recommended.</del>Use TLSv1 only! SSLv3 has proven to be vulnerable to attacks (i.e. POODLE)._

### OpenSSH

Just configure your servers to accept SSH2. The default is to accept SSH1 and SSH2 so this is a must-have configuration since SSH1 is weak.

{{< highlight apacheconf >}}
Protocol 2
{{< /highlight >}}

That&#8217;s all! If you are using something to make your connection secure, make sure that this &#8220;something&#8221; is also secure! 

I&#8217;ll be writing more about <acronym title="Man-in-the-middle">MITM</acronym> attack along the week.

 [1]: http://delicious.com/jseidl/tomcat+ssl "Tomcat + SSL tags @ jseidl's delicious"
