---
title: Batch-installing on Windows platforms with Ninite
author: Jan Seidl
type: post
date: 2010-07-05T19:11:59+00:00
url: /posts/batch-installing-on-windows-platforms-with-ninite/
categories:
  - Configuration Management
tags:
  - batch installation
  - configuration management
  - lumension
  - ninite
  - puppet

---
Today a friend introduced me that great tool called [Ninite][1]. It&#8217;s a batch-installer which comes with the most popular Windows <acronym title="Free / Open Source Software">FOSS</acronym> available in the wild and saves you from downloading and manually installing them. 

Ninite also skips installation screens (and their <acronym title="End-User License Agreement">EULA</acronym>) and it&#8217;s perfect when you must setup several stations or you have constantly some station setup to do. Although Ninite ISN&#8217;T a configuration management tool but it is a good time-saver for smaller environments.

<img src="https://wroot.org/wp/wp-content/uploads/2010/07/ninite.jpg" alt="" title="The Ninite Installer" width="400" height="254" class="aligncenter size-full wp-image-88" srcset="https://wroot.org/wp/wp-content/uploads/2010/07/ninite.jpg 400w, https://wroot.org/wp/wp-content/uploads/2010/07/ninite-300x190.jpg 300w" sizes="(max-width: 400px) 100vw, 400px" />

Ninite Pro Account gets it&#8217;s a little smarter: Downloads software according to your processor architecture and system language, say &#8216;No&#8217; to toolbars, offline installation and some other goodies.

It maybe a solution while we still wait for [Puppet][2] to support Windows stations, being a good <acronym title="Free / Open Source Software">FOSS</acronym> initiative to great market tools like [Lumension&#8217;s Endpoint Manage Security Suite][3].

 [1]: http://ninite.com/
 [2]: http://www.puppetlabs.com/puppet/introduction/
 [3]: http://www.lumension.com/Products/Endpoint-Management-Security-Suite.aspx