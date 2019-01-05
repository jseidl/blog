---
title: GoldenEye 2.1 released with even more randomness
author: Jan Seidl
type: post
date: 2014-02-21T18:32:14+00:00
url: /posts/goldeneye-2-1-released-with-even-more-randomness/
categories:
  - Application Security
  - Penetration Testing
tags:
  - ddos
  - dos
  - goldeneye
  - python

---
Recently I&#8217;ve discovered that GoldenEye got his <a href="http://services.netscreen.com/documentation/signatures/HTTP%3ADOS%3AGOLDENEYE-DOS.html" target="_blank">first signature from a big vendor</a>.

That&#8217;s funny since the main GoldenEye objective is to be signature-proof due its randomness. I&#8217;ve done a quick search on the internet and found the signature update link for their products, downloaded, located that one mentioned above.

It was a very crude signature, as expected:

{{< highlight xml >}}
<Pattern>\[Keep-Alive: 1(1\d|20)\]</Pattern>
<Pattern>\[Cache-Control: (must-revalidate, )?no-cache\].*</Pattern>
<Pattern>\[http://(www\.(google|usatoday)|engadget\.search\.aol)\.com/(search)?(/results)?\?q\].*</Pattern>
<Pattern>.*\[(Mozilla/\d\.\d|Opera 9.80) \((Windows|X11|compatible); U?; (Linux|Windows NT \d\.\d|MSIE \d\.\d)\].*</Pattern>
{{< /highlight >}}

Mostly are the only static texts GoldenEye had (since this update ;)). These are mostly leftovers from <a href="http://www.sectorix.com/2012/05/17/hulk-web-server-dos-tool/" target="_blank">Barry&#8217;s HULK</a>, which GoldenEye was spawned from.

## Changelog

No worries, this new patch include the following changes:

  * Referer strings from search engines now only domain part hardcoded (rest <querystring, path> is generated)
  * Referer generation function now generates even more random referers.
  * **NO MORE HARDCODED USER AGENTS.** I admit, hardcoded user agents were lame. There&#8217;s now a User-Agent Generator function that will generate <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html" target="_blank">RFC-2616</a> compliant user-agent strings.
  * External User-Agent List Support: As the generator function may generate UAs for inexistent browser version + plugin version combinations, you can now supply your own list of User-Agents (one per line &#8211; text file) via the `-u` flag.
  * Besides `no-cache` I&#8217;ve added the directive `max-age=0` that does basically the same thing. GoldenEye will chose one of them during the strike request.
  * More random keepalive values: They&#8217;re 110-120 (legacy), now they&#8217;re random 1-1000
  * User-Agent lists: I&#8217;ve added a `res` directory for external resources. Multiple text files were placed there with user agents from different platforms.
  * Utilities! Now the getuas.py scrapes (requires <a href="http://www.crummy.com/software/BeautifulSoup/" target="_blank">BeautifulSoup</a>) http://www.useragentstring.com/ URLs.

## About the User-Agent generation algorithm

The user-agent string follows the following format: **Mozilla/\[version\] (\[system and browser information]) [platform\] ([platform details]) [extensions]**

I have a python dictionary with <acronym title="Operating System">OS</acronym>-specific values and Platform-specific (Webkit, Gecko, Internet Explorer) values. There are many options for each one. Mostly generated on the fly thanks to python&#8217;s dynamic lists generation.

Here&#8217;s an example of the property generation

{{< highlight python >}}
'webkit': {
    'name': [ 'AppleWebKit/%d.%d' % (random.randint(535, 537), random.randint(1,36)) for i in range(1, 30) ],
    'details': [ 'KHTML, like Gecko' ],
    'extensions': [ 'Chrome/%d.0.%d.%d Safari/%d.%d' % (random.randint(6, 32), random.randint(100, 2000), random.randint(0, 100), random.randint(535, 537), random.randint(1, 36)) for i in range(1, 30) ] + [ 'Version/%d.%d.%d Safari/%d.%d' % (random.randint(4, 6), random.randint(0, 1), random.randint(0, 9), random.randint(535, 537), random.randint(1, 36)) for i in range(1, 10) ]
},
{{< /highlight >}}

Upon program start, it will generate N random values and populate the python list. As lists can be easily joined with the `+` operator, this makes dynamic list generation a charm. The same goes to <acronym title="Operating System">OS</acronym>-specific values

{{< highlight python >}}
'ext': [ 'Intel Mac <acronym title="Operating System">OS</acronym> X %d_%d_%d' % (random.randint(9, 11), random.randint(0, 9), random.randint(0, 5)) for i in range(1, 10) ]
{{< /highlight >}}

Any effort now to block our user-agents will block legitimate traffic also :)

## About referer generation

In the previous GoldenEye versions, referers were crude and simple, like search engine search urls with some random parameter. Now referers are generated like request urls:

  * Random PATH (/Hiad727ja)
  * Random QueryString key and value names
  * Random QueryString key and value quantity
  * Random QueryString presence

As it was before, referer presence is also random.

I think that covers all the changes for this version. 

[Download][1], test (please, not on other people&#8217;s servers) and report!

 [1]: https://github.com/jseidl/GoldenEye
