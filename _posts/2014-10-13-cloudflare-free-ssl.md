---
layout:     post
title:      Cloudflare's Free SSL
date:       2014-10-13 12:31:19
categories:
  - cloudflare
  - ssl
  - encryption

---

A couple of weeks ago Cloudflare began offering SSL with their free plan. This is a pretty generous thing to do. It means that I can host this blog on Github, who are also being very generous, and get SSL and content delivery network features from Cloudflare--all for free!

<!-- more -->

Cloudflare's [announcement](http://blog.cloudflare.com/introducing-universal-ssl/):

> The team at CloudFlare is excited to announce the release of Universal SSL™. Beginning today, we will support SSL connections to every CloudFlare customer, including the 2 million sites that have signed up for the free version of our service.

Further, recently [Google said](http://googlewebmastercentral.blogspot.ca/2014/08/https-as-ranking-signal.html) that they would give a small ranking increase to SSL enabled sites:

> adding a SSL 2048-bit key certificate on your site — will give you a minor ranking boost.

Many people took that to mean "get your site SSL enabled or else" and for technical blogs like this one, about the only way people find the posts is via Google.

### Cloudflare configuration

I setup my domain in Cloudflare and had it scan my current DNS settings. Then I pointed my domain's DNS servers to use the Cloudflare nameservers.

Next, in Cloudflare's web GUI, I changed my root domain to use "CNAME Flattening" setup my domain to be an alias of the [flatsec.github.io](http://flatsec.github.io) URL (which was previously setup properly, using a CNAME file and such).

[CNAME flattening](https://support.cloudflare.com/hc/en-us/articles/200169056-CNAME-Flattening-RFC-compliant-support-for-CNAME-at-the-root)...

> CloudFlare uses CNAME Flattening to present a CNAME as an A record.

After about 24 hours Cloudflare was working and had created a shared, or universal as Cloudflare calls it, SSL certificate for ```flatlinesecurity.com```.

I also enabled and configured two-factor authentication.

### Thoughts, goals, and concerns

Like technology, information security is complicated. Very complicated. Often I encounter situations in which there is no clear cut answer, and no way to make everyone happy and secure. I try to do what's reasonable. Implementing "resonable security" means performing some due dilligence with regards to risk; it means getting a good understanding of what's actually going on; it means thinking about what you're doing and why you're doing it, and that is what I like about infosec.

These are some of the thoughts, goals, and concerns I have about using Cloudflare and their free SSL:

1. For me, having SSL is better than not having SSL
2. I'd like to support SSL, though understand I don't have any sensitive information...this is just a technical blog
2. As I have had clients ask about Cloudflare, I'd like to get some experience with it
2. Cloudflare may be able to serve my site faster
2. I am not a large organization, just one person, thus free blog hosting and free SSL is good for me (even though SSL certs are getting very affordable)
2. Cloudflare takes over my DNS (though they do support [two-factor authentication](http://blog.cloudflare.com/2-factor-authentication-now-available/), which is why I recently moved to [NameCheap](https://www.namecheap.com/) for my domains)
2. The free SSL cert is a shared one--in fact as of this moment there are two sites listed in the alternative names, flatlinesecurity.com and netty.io, I have a feeling more will be added
2. Everything is "proxied" through Cloudflare, so they can gather any metrics and data that they feel like, but I will probably use Google Analytics anyways (perhaps I shoudln't?)
2. SSL is not a "silver bullet", in fact depending on who you talk to some people think it's horribly broken
2. Cloudflare still connects to github via plain text
2. At this time, Cloudflare seems to be trying to do [the right thing](http://krebsonsecurity.com/2014/02/the-new-normal-200-400-gbps-ddos-attacks/) in terms of keeping sites online
2. CloudFlare has "apps" that can provide various features, mostly related to security of some kind. As far as I can tell the only thing that was on in my account was Scrapeshield's feature to "obfuscates email address from bots" and nothing else.
2. I don't want anything enabled that might stop people from viewing my site via Tor or VPNs or otherwise--this is something I need to do a bit more research on
2. Server Name Identification (SNI) is not universally supported, which means some browsers might fail when accessing my site via SSL (mostly old browsers on old operating systems)
2. Google might give a ranking boost to SSL sites (however small that boost is)

### Conclusion

For me, as an individual technical blogger, I think it's reasonable to use Cloudflare's free SSL service. However, what's reasonable for me in this particular situation might not be reasonable for you or your business.

Basically, I'm availing myself of several free services, including Google Analytics, Google Fonts, Disqus comments, etc, and all those services, while not costing me anything, do provide valuable information for the vendor in terms of data. At some point in the future it might make sense for me to host this site elsewhere (I do like [Digital Ocean](https://www.digitalocean.com/)), but for the time being I'm Ok with how it's setup.
