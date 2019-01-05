---
title: New version of GoldenEye WebServer DoS tool released
author: Jan Seidl
type: post
date: 2013-03-26T21:58:19+00:00
url: /posts/new-version-of-goldeneye-webserver-dos-tool-released/
categories:
  - Penetration Testing
tags:
  - dos
  - goldeneye
  - http
  - https
  - layer 7
  - tool
  - webserver

---
After the [hackers 2 hackers conference talk last year][1], some people contacted me about known Python performance issues regarding the use of threads related to the <a href="http://en.wikipedia.org/wiki/Global_Interpreter_Lock" target="_blank">GIL</a>.

Indeed the threading wasn&#8217;t performing well due the nature of GIL so I&#8217;ve rewritten the code to support <a href="http://docs.python.org/library/multiprocessing.html" target="_blank">Python&#8217;s multiprocessing module</a>. It&#8217;s a tad faster but I haven&#8217;t tested it exhaustively so if you feel the inner-beta-tester in you, let me know!

The download is available as always at the github project page at <a href="https://github.com/jseidl/GoldenEye" target="_blank">https://github.com/jseidl/GoldenEye</a> and you can read more about the tool at [the project page at this blog][2].

Please test it (ON YOUR OWN RESOURCES!!) and let me know your thoughts!

 [1]: https://wroot.org/posts/about-hackers-2-hackers-conference-9th-edition/ "About Hackers 2 Hackers Conference 9th Edition"
 [2]: https://wroot.org/projects/goldeneye/ "GoldenEye"