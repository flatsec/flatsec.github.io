---
layout:     post
title:      Ten New Open Source Security Tools
date:       2014-11-05 14:00:00
categories:
  - general

---

Large hi-tech companies like Google, Netflix, Twitter and Facebook have been open sourcing some of their internal security tools. Most recently Facebook released OSQuery. In this post I take a quick look at a few of the recently released tools by these organizations as well as other smaller projects

<!-- more -->

## tl;dr

These are in random order.

1. [Google - Nogotofail](https://github.com/google/nogotofail)
2. [Facebook - OSQuery](https://github.com/facebook/osquery)
4. [OpenBSD 5.6](http://openbsd.org)
6. [Netflix - Message Security Layer](http://techblog.netflix.com/2014/10/message-security-layer-modern-take-on.html)
7. [Twitter - BreakoutDetection](https://github.com/twitter/BreakoutDetection)
8. [SSL Split](http://www.roe.ch/SSLsplit)
9. [Drozer](https://github.com/mwrlabs/drozer)
11. [Mozilla - MozDef](https://github.com/jeffbryner/MozDef)
12. Netflix - [Scumblr](https://github.com/Netflix/Scumblr) and [Sketchy](https://github.com/Netflix/sketchy)

## Slightly more

### 1. Nogotofail

In the blog post announcing [Nogotofail](http://googleonlinesecurity.blogspot.ca/2014/11/introducing-nogotofaila-network-traffic.html) Google describes it as:

>...an easy way to confirm that the devices or applications you are using are safe against known TLS/SSL vulnerabilities and misconfigurations. Nogotofail works for Android, iOS, Linux, Windows, Chrome OS, OSX, in fact any device you use to connect to the Internet. There’s an easy-to-use client to configure the settings and get notifications on Android and Linux, as well as the attack engine itself which can be deployed as a router, VPN server, or proxy.

I haven't had a chance to play with it, but as far as I can tell from the [documentation](https://github.com/google/nogotofail/blob/master/docs/getting_started.md) ```nogotofail``` sits in between clients and sites (ie. man-in-the-middle) and can help analyze traffic, attacks, vulnerabilities, etc. SSL is getting a lot of attention these days, for the better I believe.

### 2. OSQuery

OSQuery is probably the newest entry into this "open source security software" category.

OSQuery:

>exposes an operating system as a high-performance relational database.

Like Nogotofail, I haven't had a chance to test this out yet. I had initially thought that it would present an interface to many hosts, but it seems like you run osquery on each host and then can use scheduled requests to get information across a large number of hosts. I'd love to know how osquery is actually used at Facebook. It would seem like information like this could be provided by other "single sources of truth," for example perhaps PuppetDB or a Chef server or similar.

Interestingly, Micheal Crosby wrote up a quick application that is similar but for Docker: [dockersql](https://github.com/crosbymichael/dockersql)

### 3. OpenBSD 5.6+

We can't forget one of my favorite projects, OpenBSD.

OpenBSD has done a lot of interesting work lately, such as:

* [LibreSSL](http://www.libressl.org/) - Complete with Comic sans :)
* [httpd](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/httpd.8?query=httpd) - Note, not Apache httpd, instead OpenBSD's own http server
* [relayd](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/relayd.8?query=relayd)
* [Removing loadable kernel modules](http://www.openbsd.org/faq/current.html#20141013)

### 4. Message Security Layer

Netflix recently released MSL:

>With MSL we have eliminated many of the problems we faced with HTTPS and platform integration. Its flexible and extensible design means it will be able to adapt as Netflix expands and as the cryptographic landscape changes.

As most know, SSL issues such as Heartbleed and Poodle have shown there is work to be done to shore up OpenSSL. Netflix also describes a couple of other issues they have with SSL and reasons why they decided to put considerable work and thought into MSL.

### 5. Breakout Detection

[Breakout Detection](https://blog.twitter.com/2014/breakout-detection-in-the-wild) is an open source R package from Twitter.

>Given the real-time nature of Twitter, and that high performance is key for delivering the best experience to our users, early detection of breakouts is of paramount importance. Breakout detection has also been used to detect change in user engagement during popular live events such as the Oscars, Super Bowl and World Cup.

Basically it allows analysis of time series "big data" to search for:

1. Mean shift - Sudden changes
2. Ramp ups - Gradual increases

### 6. SSL Split

[SSL Split](http://www.roe.ch/SSLsplit) is a smaller project by Daniel Rothlisberger et al.

>SSLsplit is a tool for man-in-the-middle attacks against SSL/TLS encrypted network connections. Connections are transparently intercepted through a network address translation engine and redirected to SSLsplit. SSLsplit terminates SSL/TLS and initiates a new SSL/TLS connection to the original destination address, while logging all data transmitted. SSLsplit is intended to be useful for network forensics and penetration testing.

### 7. Drozer

[Drozer](https://github.com/mwrlabs/drozer) has been out for a while. I haven't tried it out yet or read much about it, but it seems like a good tool to use to suss out Android security. As mobile [is eating the world](http://ben-evans.com/benedictevans/2014/10/28/presentation-mobile-is-eating-the-world) I should probably pay more attention to it.

### 8. MozDef

Ah, analyzing logs. MozDef is pretty new and beta-ish, but who doesn't want great ways to manage logs, especially with regards to security? There is a [video overview](https://air.mozilla.org/intern-presentations-11/) available, and [slides here](http://anthony-verez.fr/mozdef/).

>Logs are large, messy and unruly. How to manage your security with such logs? Enters MozDef, your best SIEM friend to transform those chaotic streams of texts into a reliable and manageable source of events to help you visualize your security monitoring. In this presentation we'll look into why MozDef will make your security analysis work less painful and more efficient.

### 9. and 10. Scumblr and Sketchy

Finally we have [Scumblr and Sketchy](http://techblog.netflix.com/2014/08/announcing-scumblr-and-sketchy-search.html
). For the most part, these two applications might not be that useful to a smaller organization, but I'm sure someone will find interesting things to do with them, even in companies that aren't Facebook, Google, Netflix or Twitter.

Scumblr:

>Scumblr is a Ruby on Rails web application that allows searching the Internet for sites and content of interest. Scumblr includes a set of built-in libraries that allow creating searches for common sites like Google, Facebook, and Twitter. For other sites, it is easy to create plugins to perform targeted searches and return results. Once you have Scumblr setup, you can run the searches manually or automatically on a recurring basis.

Sketchy works with Scumblr to take screenshots:

>One of the features we wanted to see in Scumblr was the ability to collect screenshots and text content from potentially malicious sites - this allows security analysts to preview Scumblr results without the risk of visiting the site directly.
