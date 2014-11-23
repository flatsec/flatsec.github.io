---
layout:     post
title:      Onion Layers vs Soft Chewy Center
date:       2014-11-23 14:00:00
categories:


---

If you've worked at all in the information or computer security industry, no doubt one or more of the following phrases, "defense in depth," "layered defense," or "onion layers," has come up.

<!-- more -->

## Defense in Depth

In theory, defense in depth is a great concept. Simply stated--we create multiple layers of security, each one as strong as the last (or stronger), and that will help to ensure that if one layer is compromised that we either contain that compromise to a particular area, or there are still additional layers to break through.

However, in practice, it seems to lead to a hard shell and a soft, chewy inside.

## Soft, Chewy Interior

What we want is for each layer to be as secure as is reasonably possible, not reduced security at each layer. What we **don't** want is a single layer--a hard shell--and then a soft, chewy interior.

![onion layers vs soft chewy center](/images/posts/onion-vs-chewy.png)

## Example: IPMI

I like to pick on IPMI. It's not that I have something personal against IPMI, it's just that as a systems administrator I always need it, and IPMI has more than its share of problems...so it's everywhere and it's [usually insecure](http://www.fish2.com/ipmi/):

>For over a decade major server manufacturers have harmed their customers by shipping
servers that are vulnerable by default, with a management protocol that is insecure by
design, and with little to no documentation about how to make things better.

Below I have a diagram of what many organizations do to enable remote access via IPMI. It's usually a VPN connection to a management network in which IPMI is available to each server. The VPN is the "hard shell," and once that barrier is breached, the management network is usually soft and chewy.

![onion layers vs soft chewy center](/images/posts/onion-ipmi.png)

Given that most BMC boards are actually small, insecure computers which are poorly thought out and almost never updated (and thus are likely full of remote exploits), they are a great target. And, of course, the BMC allows access to the BIOS of the server. It's like having physical access remotely, and it's pretty difficult to protect against physical access.

## Conclusion

I think if there is any one thing to get across in this post it's that defense in depth doesn't mean that each sub-layer should be less secure than the one before. Each layer should be as secure as is reasonably possible. Further to that, when possible, we want to contain compromises of particular layers to smaller failure domains.

In practice, organizations typically use defense in depth to allow access to a soft, chewy core. Instead they should be ensuring that there is no chewy core.
