#Cool Bugs Relevant to this Course

## Heartbleed bug (April 2014)

*Aspect of the course:* We've talked a lot about buffer overruns in C this semester and the need to program defensively.  This is a great case study in why you need to be aware of this particular aspect of C programming.

*Why was it a big deal*: 
This bug was in a code base known as OpenSSL.  SSL stands for Secure Sockets Layer and is a protocol (set of rules) by which information is securely passed around the internet.  The bug was in a crucial bit of C code which implemented this (and other) protocols.  OpenSSL is used by many, many webservers which need secure communication (17% of all those on the internet, actually) so this was a widespread problem.  This bug allowed malicious attackers to gain access to memory which could have been used to store decrypted passwords.

*How it worked:*
As always, the webcomic XKCD does a great job breaking down a complex bit of code into something understandable:
![XKCD comic](https://xkcd.com/1354/)

Essentially two computers are connected to each other (securely, we hope).  These computers occasionally send a "heartbeat request" -- just a little query asking, "Hey, are we still connected?"  There's a response of just a few bytes to this query, and the request that is sent includes the correct length of the response.  When a computer receives a request, it recieves two pieces of information: the response to be sent, and the length of that response.

The receiver then allocates a buffer to store the response in.  However, the code *did not verify* that the length of the buffer actually matched the length of the response.  Using pointer arithmetic, the receiver began at the start of the requested response, cheerfully read the number of bytes requested, and sent them back to the requester.

*C code*
```
memcpy(bp, pl, payload);
```
Yep.  That's it.  Let's break it down a bit.  `memcpy` is (as you might infer) a function which copies the contents of memory from one location to another.  The parameters are `bp` (a location in the server's memory, calculed to have enough room for `pl`), `pl` (the response requested by the sender) and `payload` (the size of the response).  Attackers could scrape extra data out of a computer's memory but simply lying about how big the payload was.

The fix was relatively straightforward:
```
/* check against size 0 heartbeats first */
if (1 + 2 + 16 > s->s3->rrec.length)
   return 0;  /* silently discard request */

/* read out of memory here but don't return it until we
   do further checks */
hbtype = *p++;
n2s(p, payload);

/* check to make sure requested size of the payload (plus some overhead)
   isn't larger than it shoud be */
if (1 + 2 + payload + 16 > s->s3->rrec.length)
    return 0;  /* silently discard per RFC 6520 sec. 4 */

/* now actually assign the payload contents from memory */
pl = p;
```

*Readings:*
* https://www.seancassidy.me/diagnosis-of-the-openssl-heartbleed-bug.html
* https://www.csoonline.com/article/3223203/vulnerabilities/what-is-the-heartbleed-bug-how-does-it-work-and-how-was-it-fixed.html
* https://gizmodo.com/how-heartbleed-works-the-code-behind-the-internets-se-1561341209

## Apple's goto fail (2014)
*Aspect of the course:*
This is a really simple C programming bug, but it demonstrates the need for a solid testing infrastructure and following coding conventions.

*Why was it a big deal?:* Like the previous bug, this one occured in some security software crucial to the Internet's functionality.  It occured in Apple's SecureTransport code (their version of SSL/TLS, similar to OpenSSL). Apple uses this library for most of it security checking, and the code is native to the iOS operating system.  This meant that the bug affected virtually all iOS devices including iPads, iPhones, iPod Touches, and some Mac computers running OSX.

Unlike the previous bug, this one impacted whether or not a website was susceptible to a "Man in the Middle" attack (MitM).  This bug allows a computer to insert itself in the middle (hence, the name) of communication between a client (your computer) and a server (let's say google.com).  You would think you're visiting google.com when in fact, you're only visiting a spoofed website designed to look like google.com.  

*How it worked:*
This problematic code executes when the device connects to a secure website (one with `https` in the URL), and through a series of steps, the website proves that it is, in fact, who it says it is via web certificates.  The code should have rejected websites which couldn't pass the tests and only authenticated those which could.

*C code:*

See if you can spot the error:
```
...
	if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
		goto fail;
	if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
		goto fail;
		goto fail;
	if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
		goto fail;
 ....
  fail:
	  SSLFreeBuffer(&signedHashes);
	  SSLFreeBuffer(&hashCtx);
	  return err;
```
In this case, `goto fail` works like an unconditional branch instruction in the assembly language we've studied this semester.  When that statement executes, the code unconditionally branches to the label `fail` and proceeds from there.  The variable `err` returns the error code from the method.  It should be 0 if everything is fine and the device can trust that the website is who it says it is.  Because of the extra ```goto fail``` statement, the last check is skipped, even if it should have failed the test.

Through unit testing, this bug would have been easy to catch.  It's important to write both negative test cases (does the code correctly react to a failing scenario) as well as positive ones (does the code to the right thing).

There was also a compiler flag which would have caught the unreachable code occuring after the 2nd/extra ```goto``` statement.  Specifically the `-Wunreachable-code` should work for both `gcc` and `Clang`, two popular and widely used compilers.  Interestingly, the `gcc` developers quietly removed this option years ago due to instability.  However, the option continues to be accepted with no outward warnings that the checking IS NOT occurring!

We've also talked about coding conventions and how important they can be to a common understanding among programmers.  It's worth noting that many professional coding standards require braces around all if statements, including one liners like those above.  For example, [the Mozilla guidlines](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Coding_Style#Naming_and_Formatting_code) (makers of Firefox among other software) specifically state:
> Always brace controlled statements, even a single-line consequent of if else else. This is redundant, typically, but it avoids dangling else bugs, so it's safer at scale than fine-tuning.




*Readings:*
* https://www.imperialviolet.org/2014/02/22/applebug.html
* https://www.wired.com/2014/02/gotofail/
* https://dwheeler.com/essays/apple-goto-fail.html
* https://www.theguardian.com/technology/2014/feb/25/apples-ssl-iphone-vulnerability-how-did-it-happen-and-what-next

## Intel's Floating Point Bug (2016)
