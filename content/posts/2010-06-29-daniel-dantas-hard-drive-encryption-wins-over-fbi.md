---
title: Daniel Dantas’ hard drive encryption wins over FBI
author: Jan Seidl
type: post
date: 2010-06-29T14:58:13+00:00
url: /posts/daniel-dantas-hard-drive-encryption-wins-over-fbi/
categories:
  - Encryption
tags:
  - aes
  - bruteforce
  - encryption
  - fbi
  - nic
  - truecrypt

---
Just read it on Slashdot that [FBI failed to break encryption of Daniel Dantas&#8217;s hard drives][1].

Daniel Dantas is a brazillian banker involved in financial frauds caught by the federal police in July 2008. The 5 hard drives siezed by the Federal Police at his apartment were cyphered with the top-notch AES encryption algorithm with a 256-bit key. Two softwares were used to manage the encryption and one of them was the widely used and known open-source tool [TrueCrypt][2]. 

These drives keep secrets that can fully compromise the financial fraud and bury Dantas and other involved in jail for a long time. Both agencies (FBI and NIC) failed to crack Dantas&#8217; key. They tried bruteforce attacks for over 5 months but didn&#8217;t achieved success.

[According to G1][3] (Brazillian news site) 

> According to the report, the fed only requested help from USA in early 2009, after experts from the National Institute of Criminology (INC) failed to decode the passwords on the hard drives. The government has no legal instrument to compel the manufacturer of the American encryption system or Dantas to give the access codes.

Since the data on drives can bury Dantas, I think that he won&#8217;t be giving away the key. Since it&#8217;s a 64-character length key, I don&#8217;t think that he had it memorized. I even wonder that has been destroyed.

[<img src="https://wroot.org/wp/wp-content/uploads/2010/06/security.png" alt="&quot;Security&quot; XKCD #538 issue comic" title="&quot;Security&quot; XKCD #538 issue comic" width="448" height="274" class="aligncenter size-full wp-image-76" srcset="https://wroot.org/wp/wp-content/uploads/2010/06/security.png 448w, https://wroot.org/wp/wp-content/uploads/2010/06/security-300x183.png 300w" sizes="(max-width: 448px) 100vw, 448px" />][4]

Just for fun, I used the [Mandylion Labs Brute-force attack estimator][5] and got to the following scenario: If 100.000 machines where used in the brute force it would still take **6,42E+106 years** (sorry, no room for such zeroes) to crack the password.

Keep trying, feds! I wish you best luck (or get a $5 wrench)!

 [1]: http://yro.slashdot.org/story/10/06/26/1825204/FBI-Failed-To-Break-Encryption-of-Hard-Drives
 [2]: http://www.truecrypt.org/
 [3]: http://g1.globo.com/English/noticia/2010/06/not-even-fbi-can-de-crypt-files-daniel-dantas.html
 [4]: http://www.xkcd.org/538/
 [5]: http://www.mandylionlabs.com/PRCCalc/BruteForceCalc.htm