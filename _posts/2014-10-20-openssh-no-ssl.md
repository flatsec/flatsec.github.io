---
layout:     post
title:      OpenSSH without OpenSSL
date:       2014-10-20 14:00:00
categories:
  - ssh

---

OpenSSH supports being compiled **without** OpenSSL. OpenSSL is a lot of code, and I think that I'm pretty safe in saying more code more bugs, so if we remove a bunch of code from OpenSSH then we should have less bugs. In theory.

<!-- more -->

<span style="color:red">_NOTE: This just covers compiling ssh without ssl. Configuring it after compiling and installing is not written up yet. Once I get that working I'll come back and update this post._</span>

### New curves and keys

OpenSSH has recently [added](http://www.openssh.com/txt/release-6.5) some new elliptic curve ciphers and key formats. Some of this is Dr. Daniel Bernstein's work. See [SUPERCOP](http://bench.cr.yp.to/supercop.html), [curve25519](http://cr.yp.to/ecdh.html), etc, for more information.

>ssh(1), sshd(8): Add support for key exchange using elliptic-curve Diffie Hellman in Daniel Bernstein's Curve25519. This key exchange method is the default when both the client and server support it.

> ssh(1), sshd(8): Add support for Ed25519 as a public key type. Ed25519 is a elliptic curve signature scheme that offers better security than ECDSA and DSA and good performance. It may be used for both user and host keys.

Because of these changes, OpenSSH can be used [without ssl](http://article.gmane.org/gmane.os.openbsd.cvs/130612).

>make compiling against OpenSSL optional (make OPENSSL=no); reduces algorithms to curve25519, aes-ctr, chacha, ed25519; allows us to explore further options; with and ok djm

### Compiling without OpenSSL

I am using OpenBSD 5.5 (5.6 should be out soon), and OpenSSH 6.7. There are instructions [here](http://www.openssh.com/openbsd.html) to compile ssh. The only change I'll be making is to set ```OPENSSL=no``` when make is run.

First we download all the source code. I'm downloading the src.tar.gz file to get the entire OpenBSD source. I don't think this step is actually necessary, but to follow the official instructions to compile OpenSSH it is, so I'm doing that.

```bash
# cd /usr/src
# wget ftp://ftp.openbsd.org/pub/OpenBSD/OpenSSH/openbsd55_6.7.patch
# wget http://ftp.openbsd.org/pub/OpenBSD/5.5/src.tar.gz
# wget ftp://ftp.openbsd.org/pub/OpenBSD/OpenSSH/openssh-6.7.tar.gz
# tar zxvf src.tar.gz  
# cd usr.bin
# tar xvsfz /openssh-6.7/ssh/ /usr/src/openssh-6.7.tar.gz  
# cd ssh
```

Now patch it. It's just a one line patch for OpenBSD 5.5.

```bash
# patch -p0 < /usr/src/openbsd55_6.7.patch  
Hmm...  Looks like a unified diff to me...
The text leading up to this was:
--------------------------
|--- umac.c     Mon Oct  6 17:15:27 2014
|+++ umac.c     Mon Oct  6 17:16:38 2014
--------------------------
Patching file umac.c using Plan A...
Hunk #1 succeeded at 64.
done
```

Finally we can run make.

```bash
# make OPENSSL=no
```

Now that it's compiled we can see that it's not using OpenSSL.

```bash
# ./ssh/ssh -V
OpenSSH_6.7, without OpenSSL
```

To install, use ```make install```.

```bash
# make install
```

If you installed openssh, then restart sshd.

```bash
# kill -QUIT `cat /var/run/sshd.pid`
# /etc/rc.d/sshd start
```

### Commands and options

I found a few new (to me) interesting commands and options while working with OpenSSL 6.7.

```ssh -Q``` has a few options to check supported key types, exchanges, ciphers, etc.

```bash
# ssh -Q kex
curve25519-sha256@libssh.org
```

```ssh-keyscan``` was also useful.



### Conclusion

While compiling without OpenSSL was straightfoward, I'm struggling with the configuration. Mostly I just need to get a better understanding of the cipher, exchange, and key formats that have recently been added. So apologies on that front--I'll keep digging into this. This is a good opportunity to get back into "ssh shape" so to speak.

Also, the OpenBSD project is working hard on [LibreSSL](http://www.libressl.org/) which is a fork of OpenSSL, so eventually OpenSSH will use that. But for now it's kind of fun to try out OpenSSH without any ssl at all.

I think this could be useful in situations where you are using many OpenBSD servers and a configuration management system that uses ssh to connect, ala [Ansible](http://ansible.com). Other configuration management systems typically rely on OpenSSL certificates, systems such as Chef and Puppet. So if there are OpenSSL vulnerabilities, which there are, then these systems can also have vulnerabilities. Given the recent heartbleed issue with OpenSSL, I think we all know that it's a large codebase that hasn't had the attention it needs. I think it will get that attention now, but it's going to take time to clean it all up. In the meantime, using ssh without ssl is an interesting option.
