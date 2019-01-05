---
title: Anatomy of the W32.Stuxnet SCADA threat
author: Jan Seidl
type: post
date: 2010-08-06T13:43:41+00:00
url: /posts/anatomy-of-the-w32-stuxnet-scada-threat/
categories:
  - Malware
tags:
  - ida pro
  - malware
  - scada
  - stuxnet
  - wincc
  - windows

---
[Symantec released an in-depth analysis of W32.Stuxnet][1], reviewed through IDA-PRO. The analysis shows off the staged infection process, the unusual injection of legitimate services instead of issuing LoadLibrary calls, core encryption and exported functions. 

[Tofino Security][2] (specialized on <acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> security) has released [an excellent white paper on the case][3] (requires [free] registration). If you are a Malware Researcher / Reverse Engineer or just curious, I totally recommend it.

There&#8217;s code, great explanation, images, graphs and such explaining it all the process from infection, rooting and the propper malware code.

 [1]: http://www.symantec.com/connect/pt-br/blogs/w32stuxnet-installation-details
 [2]: http://www.tofinosecurity.com/
 [3]: http://www.tofinosecurity.com/professional/siemens-pcs7-wincc-malware