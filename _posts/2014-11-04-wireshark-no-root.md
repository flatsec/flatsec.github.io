---
layout:     post
title:      Run Wireshark without Root on Ubuntu
date:       2014-11-04 14:00:00
categories:
  - general

---

In this post I take a quick look at how to run Wireshark without being root.

Getting "promiscuous" access to a network interface on Linux requires root privileges. Running packet captures as root are dangerous. Ubuntu even has tcpdump covered when using apparmor. Why? Because malicious traffic could break tcpdump or wireshark or whatever is listening on the interface and then potentially have remote access as the same use running the dump...which is root.

<!-- more -->

## Warning!

I like [this warning](http://packetlife.net/blog/2010/mar/19/sniffing-wireshark-non-root-user/):

>WIRESHARK CONTAINS OVER ONE POINT FIVE MILLION LINES OF SOURCE CODE. DO NOT RUN THEM AS ROOT.

## Setup wireshark to use as non-root

First, install wireshark.

```bash
curtis$ sudo apt-get install wireshark -y
```

Then we can reconfigure. I imagine there is a way to do this in one step, but I haven't looked it up. Let me know in the comments if there is.

```bash
curtis$ sudo dpkg-reconfigure wireshark-common
```

This dialog will pop up. Select ```yes``` and hit enter.

![wireshark reconfigure](https://raw.githubusercontent.com/flatsec/flatsec.github.io/master/images/posts/wireshark-group.png)

There will now be a wireshark group in ```/etc/groups```.

```bash
curtis$ grep wireshark /etc/group
wireshark:x:129:
```

Add your user to it.

```bash
curtis$ adduser curtis wireshark
curtis$ grep wireshark /etc/group
wireshark:x:129:curtis
```

Relogin to have access to the new group, or use ```su``` and relogin that way in this particular terminal. Then run ```wireshark```.

```bash
curtis$ su - curtis
curtis$ wireshark
```

Now you should be able to start a capture.

![wireshark interface](https://raw.githubusercontent.com/flatsec/flatsec.github.io/master/images/posts/wireshark-interface.png)

## Ok, but how does it have root privs on the interface?

Linux capabilities. Notice ```dumpcap``` in the list.

```bash
curtis$ /sbin/getcap /usr/bin/*
/usr/bin/arping = cap_net_raw+ep
/usr/bin/dumpcap = cap_net_admin,cap_net_raw+eip
/usr/bin/fping = cap_net_raw+ep
/usr/bin/fping6 = cap_net_raw+ep
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/bin/i3status = cap_net_admin+ep
/usr/bin/systemd-detect-virt = cap_dac_override,cap_sys_ptrace+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
```

Wireshark is using dumpcap to access the interface, and dumpcap has the Linux capabilities required to do just that.

```bash
curtis$ ps ax | grep "[w]ireshark\|[d]umpcap"
 9786 pts/14   Sl     0:07 wireshark
 9813 pts/14   S      0:00 /usr/bin/dumpcap -S -Z none
```

I really need to look into [Linux capabilities](https://wiki.archlinux.org/index.php/Capabilities) more. Essentially that is the technology that is allowing the development of container systems like LXC and Docker.
