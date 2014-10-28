---
layout:     post
title:      Charlie Miller's Five Line Fuzzer
date:       2014-10-27 14:00:00
categories:
  - fuzzing

---

I've been looking into fuzzing a bit, and as a first step wanted to find the simplest fuzzer I could. I'm not sure if I've actually found the simplest fuzzer, but I did find Charlie Miller's five line fuzzer that he used to find potential exploits in PDF viewers such as Adobe's product and OSX Preview. Five lines is pretty good.

<!-- more -->

### Charlie Miller

In these two quick videos, Charlie Miller, an independent security consultant who is quite famous for his ability to find and exploit vulnerabilities, discusses his "five line fuzzer" and how it was used, along with several other tools and strategies, to locate potential vulnerabilities. Miller also worked for the NSA, and has a PHd in Mathematics.

<iframe width="560" height="315" src="//www.youtube.com/embed/Xnwodi2CBws" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="//www.youtube.com/embed/lK5fgCvS2N4" frameborder="0" allowfullscreen></iframe>

### Udacity software testing class

I also found that Miller's examples were used in the [Udacity Software Testing class](https://www.udacity.com/course/viewer#!/c-cs258/l-48329872/e-48666079/m-48739060). Part of the class discusses random testing and uses Miller's simple fuzzer as a class assignment. If you [search](https://gist.github.com/search?q=charlie+miller) github gists for "Charlie Miller" you'll come up with a bunch of hits from that class. I think it's great that a software testing class took Miller's example and ran with it.

### The Five Lines

These are the five lines Miller shows in his slides.

```bash
numwrites = random.randrange(math.ceil((float(len(buf)) / FuzzFactor))) + 1
for j in range(numwrites):
  rbyte = random.randrange(256)
  rn = random.randrange(len(buf))
  buf[rn] = "%c"%(rbyte)
```

Basically we're randomly corrupting a file. That file is fed to an application, and if the application crashes maybe there is a vulnerability/exploit.

If we hop into my favorite python appplication ipython, we can mess around with this as well.

```bash
In [33]: import random

In [34]: buf = bytearray("123456")

In [35]: rbyte = random.randrange(256)

In [36]: rn = random.randrange(len(buf))

In [37]: buf[rn] = chr(rbyte)

In [38]: buf
Out[38]: bytearray(b'12\xa7456')
```

As can be seen the buf byte array is altered with the random chararacter. I'm using the built-in ```chr``` instead the ```%c``` formatting, same thing.

### Conclusion

Interestingly Miller found a few issues just using this simple fuzzer. Though his workflow was a bit more complicated than the fuzzing code itself. However, I think this is a good example of how to get started in fuzzing systems--corrupt some files, run an application on them and see what happens. I'm certainly going to mess around with it, and perhaps rewrite the simple example in Go and keep building from there. Having said that, fuzzer's can get really complicated and at some point time would be better spent using an existing fuzzing system, one that has been around for a while. There are lots of them (subject for another post).

### PS. Untrusted binaries

lcamtuf has a [recent post](http://lcamtuf.blogspot.ca/2014/10/psa-dont-run-strings-on-untrusted-files.html), or PSA as he terms it, on not running strings on untrusted binaries:

>...the bottom line is that if you are used to running strings on random files, or depend on any libbfd-based tools for forensic purposes, you should probably change your habits. For strings specifically, invoking it with the -a parameter seems to inhibit the use of libbfd. Distro vendors may want to consider making the -a mode default, too.

So, while in his video Miller shows using his computer and his kid's computer to run his tests, lcamtuf suggests that this could lead to your computer getting cracked. I would imaging most fuzzing systems do their work in a secure, separated system of some kind. Definitely suggest doing this in a throw away environment such as a virtual machine.
