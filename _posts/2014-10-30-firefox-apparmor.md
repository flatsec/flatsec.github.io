---
layout:     post
title:      Enable Apparmor for Firefox
date:       2014-10-30 14:00:00
categories:
  - apparmor
  - Firefox

---

The desktop is a battleground! Sorry, I don't usually use military metaphors but it seems apt in this case. Many security professionals now believe there is no perimeter. Most secure systems, at least ones on premise, are hidden behind layers of firewalls, intrusion detection, and other security devices. Then we cut through those layers with VPNs or backdoors from our workstation. However, the perimeter is getting difficult to define (see: cloud), or why attack it at all when you can go after the soft underbelly of a marketing employee's laptop.

In this post I just take a quick look at enabling apparmor for Firefox.

<!-- more -->

### Workstations are a security problem

I don't want to get too much into "the desktop problem." People like [Ben Hughes](www.slideshare.net/beehooze/devopsd ay-london-ben-hughes-security) of Etsy talk about modern security issues with workstations/laptops/desktops all the time. It's a problem.

### What is apparmor?

Apparmor is a mandatory access control (MAC) system. Basically it restricts what applications can do. Not all applications need to do all the things they actually can do based on general Linux discretionary access control (DAC) . By using apparmor we can reduce the number of things applications are capable of within the OS. So, if Firefox gets cracked, we can limit the damage.

SELinux and apparmor are similar, though implement MAC somewhat differently. I actually like SELinux more, but also believe that apparmor is easier to use, and, of course, I'm using Ubuntu which comes with apparmor. If you're on Fedora, CentOS, RedHat, etc, SELinux will be available by default.

### Apparmoring Firefox

We all browse the web. It's chock full of great information, information security blogs, and hilarious cats.

I would imagine that browser code, such as Firefox's, is some of the most analyzed code in existence. Yet still browsers are not perfect. So it's not a bad idea to limit the damage they can do if cracked. Hopefully by using apparmor we can limit that damage.

By default, on Ubuntu 14.10 at least, the below output shows what ```apparmor``` is enforcing. Note that it does not include the Firefox browser. Though we do see things like dhclient, evince, cups, tcpdump, etc.

```bash
curtis$ sudo apparmor_status
apparmor module is loaded.
21 profiles are loaded.
21 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/evince
   /usr/bin/evince-previewer
   /usr/bin/evince-previewer//sanitized_helper
   /usr/bin/evince-thumbnailer
   /usr/bin/evince-thumbnailer//sanitized_helper
   /usr/bin/evince//sanitized_helper
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/cups/backend/cups-pdf
   /usr/lib/lightdm/lightdm-guest-session
   /usr/lib/lightdm/lightdm-guest-session//chromium
   /usr/lib/telepathy/mission-control-5
   /usr/lib/telepathy/telepathy-*
   /usr/lib/telepathy/telepathy-*//pxgsettings
   /usr/lib/telepathy/telepathy-*//sanitized_helper
   /usr/lib/telepathy/telepathy-ofono
   /usr/sbin/cups-browsed
   /usr/sbin/cupsd
   /usr/sbin/cupsd//third_party
   /usr/sbin/tcpdump
0 profiles are in complain mode.
3 processes have profiles defined.
3 processes are in enforce mode.
   /sbin/dhclient (7414)
   /usr/sbin/cups-browsed (857)
   /usr/sbin/cupsd (5219)
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```

The profile for Firefox exists by default, but it is disabled. We can easily enable it with one command.

```bash
curtis$ sudo apt-get install apparmor-utils -y
# don't really need apparmor-utils by why not
curtis$ sudo aa-enforce /etc/apparmor.d/usr.bin.Firefox

```

Now Firefox is also being covered by apparmor.

```bash
curtis@curtis:/etc/apparmor.d$ sudo apparmor_status
[sudo] password for curtis:
apparmor module is loaded.
26 profiles are loaded.
26 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/evince
   /usr/bin/evince-previewer
   /usr/bin/evince-previewer//sanitized_helper
   /usr/bin/evince-thumbnailer
   /usr/bin/evince-thumbnailer//sanitized_helper
   /usr/bin/evince//sanitized_helper
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/cups/backend/cups-pdf
   /usr/lib/Firefox/Firefox{,*[^s][^h]}
   /usr/lib/Firefox/Firefox{,*[^s][^h]}//browser_java
   /usr/lib/Firefox/Firefox{,*[^s][^h]}//browser_openjdk
   /usr/lib/Firefox/Firefox{,*[^s][^h]}//lsb_release
   /usr/lib/Firefox/Firefox{,*[^s][^h]}//sanitized_helper
   /usr/lib/lightdm/lightdm-guest-session
   /usr/lib/lightdm/lightdm-guest-session//chromium
   /usr/lib/telepathy/mission-control-5
   /usr/lib/telepathy/telepathy-*
   /usr/lib/telepathy/telepathy-*//pxgsettings
   /usr/lib/telepathy/telepathy-*//sanitized_helper
   /usr/lib/telepathy/telepathy-ofono
   /usr/sbin/cups-browsed
   /usr/sbin/cupsd
   /usr/sbin/cupsd//third_party
   /usr/sbin/tcpdump
0 profiles are in complain mode.
3 processes have profiles defined.
3 processes are in enforce mode.
   /sbin/dhclient (9118)
   /usr/sbin/cups-browsed (857)
   /usr/sbin/cupsd (5219)
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```

I've read a few articles suggesting that this profile is disabled because it can cause issues with Firefox. I've been browsing a few days with the apparmor profile enabled, and have not encountered any issues yet. Though that doesn't mean I won't in the future, I have a feeling that many of the issues will arise when using Java or weird 3rd party plugins, which I try not to do.

### PS. Disable Java!

Apparently 90% of browser hacks come through [Java](http://www.eweek.com/security/java-primary-cause-of-91-percent-of-attacks-cisco.html).

>The Cisco 2014 Annual Security Report found that Java represented 91 percent of all Indicators of Compromise (IOCs) in 2013.
