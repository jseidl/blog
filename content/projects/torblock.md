---
title: TOR-Block
author: Jan Seidl
type: page
date: 2012-09-28T16:56:30+00:00

---
# TORBlock

TORBlock is a BASH script to automatically download the list of <acronym title="The Onion Router">TOR</acronym> exit-nodes and add them to your IPTables ruleset.

This script uses hosts advertised on <acronym title="The Onion Router">TOR</acronym>&#8217;s public exit-node lists in order to extract <acronym title="Internet Protocol">IP</acronym> addresses and roll out the rules.

TORBlock uses and (configurable) dedicated chain (default = TORBLOCK) in order to ease management and permit chain flushing on updates.

### Download

Download the latest version of <acronym title="The Onion Router">TOR</acronym>-Block directly from the [github project page][1].

 [1]: https://github.com/jseidl/torblock