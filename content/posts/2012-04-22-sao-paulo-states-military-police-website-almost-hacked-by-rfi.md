---
title: São Paulo State’s Military Police website (almost) hacked by RFI
author: Jan Seidl
type: post
date: 2012-04-22T21:00:24+00:00
url: /posts/sao-paulo-states-military-police-website-almost-hacked-by-rfi/
categories:
  - Application Security
tags:
  - deface
  - military police
  - rfi
  - são paulo

---
Today a friend shared me <a rel="nofollow noindex" href="http://www.polmil.sp.gov.br/abrirframes.asp?PAGINA=http%3A%2F%2Fwww.nova89fm.com.br%2Fwebsite%2F" title="São Paulo State's Military Police RFI" target="_blank">this link</a> which pointed to São Paulo State&#8217;s Military Police website and showed a deface-like page with a hacktivist text and a youtube video.

The first thing that came to my eyes was the URL:

<pre>http://www.polmil.sp.gov.br/abrirframes.asp?PAGINA=http%3A%2F%2Fwww.nova89fm.com.br%2Fwebsite%2F</pre>

<img src="https://wroot.org/wp/wp-content/uploads/2012/04/pmsp1.png" alt="" title="São Paulo State&#039;s Military Police &quot;hacked&quot; site" width="640" height="512" class="size-full wp-image-201" srcset="https://wroot.org/wp/wp-content/uploads/2012/04/pmsp1.png 640w, https://wroot.org/wp/wp-content/uploads/2012/04/pmsp1-300x240.png 300w" sizes="(max-width: 640px) 100vw, 640px" />

&#8216;abrirframes.asp&#8217; is &#8216;openframes.asp&#8217; and &#8216;PAGINA=&#8217; is &#8216;PAGE=&#8217; in Brazilian Portuguese. Say no more.

That&#8217;s why I&#8217;ve said &#8220;almost hacked&#8221;. The site that was actually hacked was <a href="http://www.nova89fm.com.br" rel="nofollow noindex" title="Nova89 FM Radio - Actually Hacked Website">www.nova89fm.com.br (a Brazilian FM Radio)</a> and not São Paulo State&#8217;s Military Police&#8217;s.

But São Paulo State&#8217;s Military Police website wasn&#8217;t clean at all. This &#8216;PAGINA=&#8217; (or &#8216;PAGE=&#8217;) attribute clearly accepts any <acronym title="Uniform Resource Locator">URL</acronym> and this hacked website was &#8216;tucked-in&#8217; to look like the police website was actually hacked. The proof that any <acronym title="Uniform Resource Locator">URL</acronym> can be spoofed is [that][1] (it will open a famous brazilian news portal).

This seems to be another action of script-kiddies and defacers (almost the same) using automated tools to call themself &#8216;hackers&#8217;. I&#8217;m kinda&#8217; sorry for guys like these. 

It would be better to do a decent vulnerability disclosure instead of doing such a lame hack. Those kids.

 [1]: http://www.polmil.sp.gov.br/abrirframes.asp?PAGINA=http://www.globo.com