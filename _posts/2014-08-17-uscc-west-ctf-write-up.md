---
layout: post
title: 2014 US Cyber Challenge West CTF Write-up
comments: true
---

I had quite a few questions about what my strategy was for getting so many points at this years CTF, so I though I'd give a little write-up here for the curious.  As some background, I attended USCC West last year (that was my first time doing a CTF) and also competed in [NCL](http://www.nationalcyberleague.org/index.shtml) last fall.  Both the USCC CTF and NCL used Threatspace as the CTF provider, so coming into this year's CTF I had a good idea of what it was going to be like.

This years CTF only had 3 categories: web, password cracking, and wireless analysis.  My strategy this years was to first crack password, then examine the wireless captures.  Then, if time permitted, I would start poking at the web apps.  The reason I did this is because I learned last year during the USCC/NCL CTFs that password cracking and wireless analysis was the fastest way to get big points.  Even though though web application/browser security is my specialty(I've been studying it for many years now) and I could have gained some points that way, cracking passwords and examining pcaps always proved better in time-spent vs points-gained tradeoff during NCL.  Another benefit was that while passwords were cracking, I was free to work on other things, such as setting up vulnerability scans and examining pcaps.

Password Cracking
-----------------

"Windows Passwords 1" was an unencrypted SAM registry hive.  During the CTF, I didn't know how to quickly extract the hashes.  I checked the google to see if it had any answers, but after about a minute of research, I didn't see anything.  Time is of the essence during CTFs, so I just skipped "Windows Passwords 1" all together.  I did spend about 10 minutes later in the CTF trying to find a tool to extract the hashes, but once again no luck (The closest things I've seen so far are chntpw and Registry Viewer, but they don't extract the hashes.  If anyone knows an easy way, please let me know.).

**Update:** Commenter "RC" pointed out that you can load the registry hive in Cain & Abel to extract the hashes.  Much easier!

On "Windows Passwords 2" I just copied NTLM hashes and pasted them into [hashkiller](http://www.hashkiller.co.uk/ntlm-decrypter.aspx).  HashKiller found all except the admin password, which I set aside to crack later but ultimately forgot to crack(Edit: just cracked it - USCC-UXQL-6971).

Linux passwords 1,2, and 3 were hashed using md5crypt, sha256crypt, and sha512crypt, respectively.  The difference between these hashing families and their raw counterparts(straight MD5, SHA256, and SHA512) is that a unique salt is used for each hash as well as key-stretching.  Consequently, cracking linux passwords was much slower.

I used [oclHashcat](http://hashcat.net/oclhashcat/) for cracking on two separate machines that I was remoting into.  Once machine had AMD 7970 and the other had 4 AMD R9 290x.  Linux Passwords 1 and 2 were fast enough that I could just use the rockyou dictionary with the best64 ruleset.  WIth Linux Passwords 3 I just did a single run through of rockyou.  In the end, for Linux Passwords 1 I cracked 7/10, Linux Passwords 2 I cracked 5/10, and Linux Passwords 3 I cracked 3/10. 

I did use a couple of tricks that does speed up cracking.  For one, I set the -u and -n options in hashcat to values that worked well with my videos cards.  In addition, I had prepared my dictionaries by splitting them by length (see [here](https://hashcat.net/forum/thread-2543.html) ("Essential performance note") and [here](http://hashcat.net/wiki/doku.php?id=hashcat_utils#splitlen) for details).

Wireless Analysis
-----------------

The traffic in the pcaps in Wireless 1-7 were all encrypted.  Wireless 1-5 were all encrypted using WEP and were easily broken using aircrack (command: aircrack NAME_OF_PCAP).  Wireless 6 and 7 used WPA.  For 6 and 7 I converted them to a format hashcat can use (command: aircrack <input.cap> -J <output.hccap>).  For Wireless 6 I used the rockyou yet again and cracked it very quickly.  I didn't crack wireless 7 during the competition because it took too long, but I did crack after the competition (key: USCC-PSAT-9021).

After I had the wireless keys, the captures could be decrypted in Wireshark by going to Edit - Preferences - Protocols - IEEE 802.11 and editing the decryption keys.  Then it was just a matter of examining the captures.  Most of the traffic was HTTP (tip: set a filter in Wireshark to "http"), and was a capture of an admin accessing DD-WRT's admin interface on a wireless router.  One of the flags was to get the username and password for the admin interface.  The admin interface of DD-WRT uses basic authentication, which means that the plaintext username and base64 encoded password was sent in each request (Wireshark conveniently decodes this automatically).  

A useful feature in Wireshark that I have sometimes used is File - Export Objects - HTTP.  That allows me to extract all HTTP responses from the captures and view/search responses in a different programs.  It didn't really prove fruitful in this CTF, but some cases it was easier to examine those files rather than looking at the all the responses in Wireshark.  Another useful feature is good old Ctrl - F and then finding by strings.

Web Challenges
--------------

I only web challenges I really looked at were 1,3, and 4.  Web 1 I saw had PhpMyAdmin installed and allowed me to login with username "admin" and no password.  There two default MySQL databases were there (information_schema and test), but nothing else really looked interesting, so I moved on.

When you visited Web 3 there was a directory listing with links to challenges 1-5.  Flag 1's hint was "sniff sniff", so I looked at the traffic in Burp and the flag was in the HTTP header.  

When visiting flag_02.php, you got an error saying invalid method.  I tried all the valid [HTTP methods](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods) with no luck, but then tried using FLAG as the method and it returned the flag as well as flag 3 in the HTTP headers.  I tried the OPTIONS header again(technically, it is supposed to list all valid HTTP methods), but no luck.  Looking back now, if you look at the headers in Flag 1, one of the HTTP headers was called "METHODS" and stated GET POST and FLAG were valid HTTP methods.

Flag 4 just required using Flag 3 as the HTTP method.  Flag 5 said to take the MD5 hash of flag 4 and use it as the HTTP method.  Technically, this flag has an error in it because computing the MD5 in Burp and via "echo -n USCC-RETL-8932 | md5sum" did NOT work.  The correct answer was computed using "echo USCC-RETL-8932 | md5sum".  I say this is technically NOT the md5 of flag 4 because the echo command always includes a newline character at the end(unless you use the -n option).

I was working on Web 4 when the competition ended, so I didn't actually get any flags on it.  But I was on my way to getting some due to a directory traversal vulnerability I found.  A few of the URLs in web 4 had base64 encoded parameters in the URL, one of which was original.gif.  Original.gif had a parameter named "f" that was base64 encoded.  Decoded it looked like this:

```php
a:1:{i:0;s:10:"output.gif";}
```

That looked like a serialized PHP object to me, so to confirm I created the following PHP script

```php
<?php
var_dump(unserialize('a:1:{i:0;s:10:"output.gif";}'));
?>
```

Sure enough it returned an array object with a string inside it.  So this raised a flag in my head because that means the application is vulnerable to [PHP object injection](https://www.owasp.org/index.php/PHP_Object_Injection), for one.  That vulnerability is really only useful if you know what's going on in the code, so I moved on and started tampering with the string.  As it turns out, the string could be tampered to get directory traversal by doing the following:

```php
a:1:{i:0;s:25:"../../../../../etc/passwd";}
```

Base64 encoding this and shoving it in the "f" parameter resulted in the /etc/passwd file.  I read a few of the other files on the webserver(unfortunately the web server wasn't running as root so no shadow file), but it was about this time that the competition ended, so I wasn't able to actually get any flags.

Lessons Learned
---------------

In retrospect, there's definitely room to improve:

* I completely forgot to crack the high value NTLM on Windows Passwords 2 (it literally on took me like 2 seconds to crack).  I should have also began cracking Wireless 7 earlier since it was a super high value target (it took about an hour to crack after the competition).  These two things would have easily given me an extra 16,500 points.
* Some of the stuff could have been scripted beforehand, such as extracting hashes and automatically putting them in the format hashcat requires.
* The 4 290x GPUs I was using kept overheating (despite running at 100% fan) and I had to restart cracking.  This was quite frustrating at times and led to wasted time.
* I had to look up how to enter the decryption keys in Wireshark.  While it didn't take much time, it was a few extra minutes wasted.
* I didn't check the passwords to see if there was a theme to them.  This may have led me to getting a few more.
* Nobody used the Phone Home script to get points.  If I knew how many points the phone home script would have gave me, I may have switched my strategy to target the web apps first rather than going for quick points.  Considering many of the servers had stability issues, though, I think I chose the right strategy this time.

Overall though, I was pretty happy (and surprised!) with my results.  The competition was a lot of fun and my mind is still recovering from all the information I learned during the week.  I'm looking forward to next year and to the day when someone crushes my score!