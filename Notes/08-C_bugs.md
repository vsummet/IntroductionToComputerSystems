# A Study of Bugs Relevant to this Course

## Heartbleed bug (April 2014)

*Aspect of the course:* We've talked a lot about buffer overruns in C this semester and the need to program defensively.  This is a great case study in why you need to be aware of this particular aspect of C programming.

*Why was it a big deal*: 
This bug was in a code base known as OpenSSL.  SSL stands for Secure Sockets Layer and is a protocol (set of rules) by which information is securely passed around the internet.  The bug was in a crucial bit of C code which implemented this (and other) protocols.  OpenSSL is used by many, many webservers which need secure communication (17% of all those on the internet, actually) so this was a widespread problem.  This bug allowed malicious attackers to gain access to memory which could have been used to store decrypted passwords.

*How it worked:*
As always, the webcomic XKCD does a great job breaking down a complex bit of code into something understandable:
![XKCD comic](https://imgs.xkcd.com/comics/heartbleed_explanation.png)

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
This is a really simple C programming bug, but it demonstrates the need for a solid testing infrastructure, following coding conventions, and compiler warning/error flags.

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

There was also a compiler flag which would have caught the unreachable code occuring after the 2nd/extra ```goto``` statement.  Specifically the `-Wunreachable-code` should work `Clang`, a popular and widely used compiler.  Interestingly, `gcc` seemingly has an identical flag, but developers quietly removed the functionality of this option years ago due to instability.  However, the option continues to be accepted with no outward warnings that the checking IS NOT occurring!  However, `gcc` does offer the `-Wmisleading-indentation` option which would catch the error of improper/misleading indentation as above.

We've also talked about coding conventions and how important they can be to a common understanding among programmers.  It's worth noting that many professional coding standards require braces around all if statements, including one liners like those above.  For example, [the Mozilla guidlines](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Coding_Style#Naming_and_Formatting_code) (makers of Firefox among other software) specifically state:
> Always brace controlled statements, even a single-line consequent of if else else. This is redundant, typically, but it avoids dangling else bugs, so it's safer at scale than fine-tuning.

*Readings:*
* https://www.imperialviolet.org/2014/02/22/applebug.html
* https://www.wired.com/2014/02/gotofail/
* https://dwheeler.com/essays/apple-goto-fail.html
* https://www.theguardian.com/technology/2014/feb/25/apples-ssl-iphone-vulnerability-how-did-it-happen-and-what-next

## Intel's FDIV Bug (1994)
*Aspect of the course:* Instruction Set Architectures.   In fact, `FDIV` is the x86 assembly language instruction for floating point division.  While this bug was a hardware bug, not a software one, it demonstrates the importance of understanding what results your software should be giving you before running it so that you can "sanity check" the results.  Moreover, it demonstrates the important of adhering to international standards in storage and usage of data such as the IEEE-754 standard we discussed for storing single and double precision floating point numbers. 

*Why was it a big deal?:*  
The Pentium line of processors was (at the time) a new release for Intel.  The first, the Pentium-75, was billed as the fastest, most technologically advanced processor and was marketed as a substantial upgrade from the previous line of Intel's chips, the 486s.  However, a math professor at the small school of Lynchburg College found that floating point division results did not match what he was expecting.  In short, he ran numerous calculations and found that these calculations did not match the theoretically expected values he derived via mathematical formulas.  He set out to understand why, examining his code and proofs for logic errors, writing the program in multiple languages, and running his program(s) and many different computers from different manufacturing facilities.  His findings were confirmed by other scientists and mathematicians.  Eventually, Intel was forced to admit it was a bug which they had discovered earlier but were trying to ignore.  In a series of PR gaffes, Intel first offered new (error-free) chips only to users who could prove they needed highly accurate mathematical results.  The ensuing outcry meant that Intel eventually offered new chips to anyone who asked leading to a $500 million "recall" of the chips and doing serious damage to Intel's brand (eg. see [this list of jokes at Intel's expense](http://www.columbia.edu/~sss31/rainbow/pentium.jokes.html).

Error-free calculations are extremely important, and this bug truncated calculations results.  Given that the output of a division is often fed as input to another division (in a loop or via function compounding), the error can magnify quickly, although Intel downplayed its significance as occuring in only 1 out of 9 billion independent calculations.  

*How it worked:*
An example: `4195835/3145727` would return 1.333**739068902037589** instead of the correct value of 1.333**820449136241002**, so the error could be seen after only 5 significant digits.  To more quickly compute division of floating point numbers, chip designers had implemented an Sweeney, Robertson, Tocher (SRT) algorithm which required the use of a lookup table to achieve its performance metric of generating 2 quotient bits per clock cycle.  This lookup table contained pre-calculated values for 1066 divisions which would be intermediate steps for the algorithm.  Somehow, however, 5 entries were omitted from the table.

![Graph of bad results](https://www.cs.earlham.edu/~dusko/cs63/412038a2.gif)

*Readings:*
* http://www.emery.com/nicely.htm
* https://www.mathworks.com/matlabcentral/fileexchange/1666-pentium-division-bug-documents
* https://www.techradar.com/news/computing-components/processors/pentium-fdiv-the-processor-bug-that-shook-the-world-1270773

## Ariane 5 Flight 501 (1996)
*Concepts relevant to this course:* This is primarily a story about data representation, particularly floating point vs. integer data.  However, overflow and rounding errors also prominently play a role as does the use of legacy software which is a common practice in the software industry.

*Why it was a big deal:*  It caused spacecraft to destory itself 30 seconds after liftoff.  Do I really need to go on?  The European Space Agency had been using the predecessor Ariane 4 rocket for over 15 years.  The new Ariane 5 was meant to help the ESA be a major player in international space exploration.  The Ariane 4 was extremely solid, with no failures in its launch history.  However, the Ariane 5 was significantly bigger and could thus carry a much larger payload into space.  At 30 seconds after liftoff, the rocket went horribly off course, triggering the self-destruct mechanism which destroyed the rocket (and it's 400 million dollar payload) about 2.5 miles above Earth's surface.

*How it worked:* Due to budget and time constraints the ESA reused software from the Ariane 4 in the Ariane 5.  Specifically, the navigation system software and the flight path optimization libraries were partially reused.  Just before the rocket broke apart, the Internal Reference System (the system responsible for tracking where the rocket is) sent bad data to the Flight Control System.  The control system responded to the bad data by altering the rocket's course in a way which caused parts of the rocket to break apart and the self-destruct sequence to trigger.  

The inquiry found that a 64-bit floating point number variable was cast to a signed 16-bit integer value.  However, the floating point value was too large to be stored in the 16-bit integer variable resulting in an overflow and a very small negative number.  This code had been reused wholesale from the Ariane 4.  However, the Ariane 4 was not as powerful as the Ariane 5 and thus went at slower speeds and had lower maximum velocity calculations.  As soon as the Ariane 5 surpassed the Ariane 4's capabilities (at 30 seconds post-liftoff), the software bug occurred.  There was no exception handling code and the guidance system eventually returned an error code which was only intended for debugging purposes.  This value was then passed on to the Flight Control System.

It turns out the software which caused the error wasn't essential!  From the NYTimes:
> One extra absurdity: the calculation containing the bug, which shut down the guidance system, which confused the on-board computer, which forced the rocket off course, actually served no purpose once the rocket was in the air. Its only function was to align the system before launch. So it should have been turned off. But engineers chose long ago, in an earlier version of the Ariane, to leave this function running for the first 40 seconds of flight -- a "special feature" meant to make it easy to restart the system in the event of a brief hold in the countdown.

*Readings*
* http://sunnyday.mit.edu/nasa-class/Ariane5-report.html
* https://www.nytimes.com/1996/12/01/magazine/little-bug-big-bang.html
* http://www.rvs.uni-bielefeld.de/publications/Reports/ariane.html
* https://hackernoon.com/crash-and-burn-a-short-story-of-ariane-5-flight-501-3a3c50e0e284
