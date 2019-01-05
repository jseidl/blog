---
title: Usernamer
author: Jan Seidl
type: page
date: 2012-09-28T16:54:05+00:00

---
# Usernamer

usernamer is a penetration testing tool to generate a list of possible usernames/logins for determined name (ex: John Doe Doeson) for user enumeration or bruteforcing. This tool also supports text-files with one name per line as input.

### Features

usernamer has a plugin structure that enables a series of transformations:

  * normal: Permutates given name with all surnames (if more than one) with name starting and ending (johndoedoeson,johndoesondoe,doedoesonjohn etc)
  * two_terms: Permutates given name with all surnames (if more than one) with name starting and ending but it will output a two-termed login (johndoe, doejohn, johndoeson etc)
  * one_term: Permutates all name tokens (first name and surnames) and generates single terms usernames (john, doe, doeson)
  * dotted\_two\_terms: Permutates given name with all surnames (if more than one) with name starting and ending but it will output a two-termed login dot-separated (john.doe, doe.john, john.doeson etc) 
  * normal\_abbreviated: Generates abbreviated versions of the &#8216;normal&#8217; and &#8216;two\_terms&#8217; plugins (jdoe, johnd, jd etc)

### Examples

{{< highlight bash >}}
$ ./usernamer.py -n 'John Doe' -l
jdoe
dj
john.doe
johnd
doej
johndoe
doe
doe.john
djohn
jd
john
{{< /highlight >}}

### Download

Download the latest version of usernamer directly from the [github project page][1].

 [1]: https://github.com/jseidl/usernamer
