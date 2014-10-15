---
layout:     post
title:      Security Means Tradoffs
date:       2014-10-15 14:00:00
categories:
  - basics
  - psychology

---

There's no way around it. Implementing information security means tradeoffs, and that is something that few people are willing to understand, or accept. But it's true. If you want additional security you have to give something up, usually convenience. That's just how it works. It's a concept that we need to accept psychologically, and also something we need to think about rationally.

<!-- more -->

Bruce Schneier has written about how [security requires tradeoffs](https://www.schneier.com/essays/archives/2008/01/the_psychology_of_se.html):

> Security is a trade-off...[this] is a notion critical to understanding the psychology of security. There's no such thing as absolute security, and any gain in security always involves some sort of trade-off

Schneier goes on to mention other things that security costs us, ie. what we have to trade, on occasion, for security:

* money
* time
* convenience
* capabilities
* liberties (careful with this one)
* more...

I want to make very clear that this doesn't mean we have to trade everything to try to obtain some sort of absolute security. Rather, what we need to do is make conscious decisions about what tradeoffs we are going to make based on what we think, after careful consideration, the real risks are. In fact there are often times we **should not** make the tradeoff, especially with regards to overreacting to unlikely risk.

### An example: two-step authentication

There are many big examples I could use (such as riding a bike to work versus driving in a car) but I think what I'd like to talk about is two-factor authentication, perhaps better phrased as "two-step verification."

The article "[How I Lost My $50,000 Twitter Username](https://medium.com/@N/how-i-lost-my-50-000-twitter-username-24eb09e026dd)" is still one of the more fascinating articles I've read on real-life online security.

The author writes:

>I had a rare Twitter username, @N. Yep, just one letter. I’ve been offered as much as $50,000 for it. People have tried to steal it. Password reset instructions are a regular sight in my email inbox. As of today, I no longer control @N. I was extorted into giving it up.

Basically, what happened in this situation was that Godaddy allowed the attacker to take over _@N_'s account and therefore control the email for that domain. _@N_ had used that custom domain to register his twitter account. Thus, because almost every online service provides a password reset via email, inlcuding twitter, the attacker was able to request a password reset of the twitter account, and because he controlled the email for that domain he received the reset message and was able to reset the password to one only he knows.

In the comments for that article there are a lot of suggestions as to what could have been done to reduce the likelyhood of this attack, which one commenter called a "chain hack", which I kind of like the name of, to work.

* Don't use "vanity" or custom domains to as the email registered with social media sites
* Use a gmail account, noting that you can do things like using a [plus sign after your account name](https://support.google.com/mail/answer/12096?hl=en), eg. ```your.email+somesocialsite@gmail.com```, to allow use of multiple twitter accounts from one gmail account, or to keep better track of emails.

>Gmail doesn't offer traditional aliases, but you can receive messages sent to your.username+any.alias@gmail.com. For example, messages sent to jane.doe+notes@gmail.com are delivered to jane.doe@gmail.com.

* Use two-step authentication
* Understand that even doing the above the vendors may still provide someone else access to your account via social engineering and poor security practices
* Increase the TTL of MX entries to increase the time it would take an attacker to change the email servers, in hopes that that time would be enough to stop them from obtaining your other accounts
* Understand that the service might have backup options that allow circumventing your carefully secured account (eg. your mother's maiden name is public knowledge, not a secret)

### But what are the tradeoffs?

So I got a little out of hand there. Social networking account security, well any account security, is pretty complicated and quite interesting.

But the point I wanted to make was that there are tradeoffs when increasing security, and I specifically wanted to take a look at what tradeoffs there are for using two-step authentication using [Google Authenticator](http://www.google.ca/landing/2step/) on your smartphone.

### Google Authenticator: real versus perceived tradeoffs

What people **think** the tradeoffs are:

* You have to have your phone available, and get that darn number from the app, every time you login (how annoying!)
* If you lose your phone you will be locked out of your account...forever!

What the real tradeoffs are:

* If you use the same device to login, about once a month (by default, it's configurable) you'll have to login using two-step.
* If you access your account from a new device, you'll have to use Google Authenticator, but that's the point--an attacker would have to do the same
* You have print off and store backup codes for Google Authenticator so that in the case you lose your phone you can still access your account

That's really about it. Occasionally login using your password and a number generated by the Google Authenticator application, and also somehow safely store a print-off of backup login credentials.

I think that, in this particular example, the tradeoffs are worth the increased security. What's more, that the perceived tradeoffs are actually much different than reality.

### Conclusion

Security is a difficult and complicated thing, not only technically but psychologically as well.

In the end, I think it's important to do three things:

1. Accept that security requires tradeoffs
2. Understand that perceived tradeoffs are often not the same as real tradeoffs
3. Be cognizant as to what is being given up, or traded, for security (because it's not always worth it)

Finally, sorry, I can't help but include this quote:

>"Those who would give up essential Liberty, to purchase a little temporary Safety, deserve neither Liberty nor Safety." <br> -- Benjamin Franklin
