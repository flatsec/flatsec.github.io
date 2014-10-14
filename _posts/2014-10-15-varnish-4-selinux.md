---
layout:     post
title:      Varnish 4.0 on CentOS 6 with SELinux
date:       2014-10-15 12:31:19
categories:
  - selinux
  - sysadmin

---

As of this writing, Varnish 4.0 won't run on a CentOS 6.x server when SELinux is enforcing.

<!-- more -->

There is probably nothing more common for a RedHat/CentOS admin to do than disabling SELinux. But we can't **always** disable SELinux.

Below you can see we have a CentOS 6.5 server and SELinux is enforcing.

```bash
[root@cache sysconfig]# cat /etc/redhat-release
CentOS release 6.5 (Final)
[root@cache sysconfig]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /selinux
Current mode:                   enforcing
Mode from config file:          error (Success)
Policy version:                 24
Policy from config file:        targeted
```

But varnish 4.0 can't be started.

```bash
[root@cache sysconfig]# service varnish start
Starting Varnish Cache:                                    [FAILED]
[root@cache sysconfig]# rpm -qa | grep varnish
varnish-4.0.2-1.el6.x86_64
varnish-libs-4.0.2-1.el6.x86_64
```

If I disable SELinux and reboot varnish will start.

There are a few error logs in ```/var/log/audit/audit.log```, and I'll drop them in here just for google searches.

```bash
[root@cache audit]# grep AVC /var/log/audit/audit.log
type=AVC msg=audit(1413316302.956:4): avc:  denied  { chown } for  pid=1050 comm="varnishd" capability=0  scontext=system_u:system_r:varnishd_t:s0 tcontext=system_u:system_r:varnishd_t:s0 tclass=capability
type=AVC msg=audit(1413316303.785:5): avc:  denied  { fowner } for  pid=1050 comm="varnishd" capability=3  scontext=system_u:system_r:varnishd_t:s0 tcontext=system_u:system_r:varnishd_t:s0 tclass=capability
type=AVC msg=audit(1413316303.785:5): avc:  denied  { fsetid } for  pid=1050 comm="varnishd" capability=4  scontext=system_u:system_r:varnishd_t:s0 tcontext=system_u:system_r:varnishd_t:s0 tclass=capability
type=AVC msg=audit(1413316832.823:52): avc:  denied  { chown } for  pid=1573 comm="varnishd" capability=0  scontext=unconfined_u:system_r:varnishd_t:s0 tcontext=unconfined_u:system_r:varnishd_t:s0 tclass=capability
type=AVC msg=audit(1413316881.454:59): avc:  denied  { chown } for  pid=1614 comm="varnishd" capability=0  scontext=unconfined_u:system_r:varnishd_t:s0 tcontext=unconfined_u:system_r:varnishd_t:s0 tclass=capability
type=AVC msg=audit(1413317112.992:65): avc:  denied  { chown } for  pid=1704 comm="varnishd" capability=0  scontext=unconfined_u:system_r:varnishd_t:s0 tcontext=unconfined_u:system_r:varnishd_t:s0 tclass=capability
```

There's a [bug ticket](https://bugzilla.redhat.com/show_bug.cgi?id=1083111) for this problem and it was fixed in Fedora 19. But I'm not running Fedora 19, I'm running CentOS 6.5.

### SELinux troubleshooting

I thought this might be a good time to learn about SELinux. There's a good document [here](http://wiki.centos.org/HowTos/SELinux) that I'm following to create a policy that will allow varnish 4 to startup and work properly.

First let's install some packages.

```bash
[centos@cache ~]$ sudo yum install -y policycoreutils-python setroubleshoot-server
```

Now we can translate the audit log into something human readable.

```bash
[root@cache ~]# sealert -a /var/log/audit/audit.log > audit.txt
```

There's a few alerts in there, but they basically look like this:

```bash
found 3 alerts in /var/log/audit/audit.log
--------------------------------------------------------------------------------

SELinux is preventing /usr/sbin/varnishd from using the chown capability.

*****  Plugin catchall (100. confidence) suggests  ***************************

If you believe that varnishd should have the chown capability by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# grep varnishd /var/log/audit/audit.log | audit2allow -M mypol
# semodule -i mypol.pp
```

So I ran this:

```bash
[root@cache ~]# grep varnishd /var/log/audit/audit.log | audit2allow -M varnishpol
```

Which created a couple of files, one data file and one configuration file. Let's look at the configuration file.

```bash
[root@cache ~]# cat varnishpol.te

module varnishpol 1.0;

require {
	type varnishd_t;
	class capability { fowner chown fsetid };
}

#============= varnishd_t ==============
allow varnishd_t self:capability { fowner chown fsetid };
```

As can be seen above, varnishd_t is getting three new capabilities: fowner, chown, fsetid.

Using the ```varnishpol.pp``` file I can load the varnish policy.

```bash
[root@cache ~]# semodule -i varnishpol.pp
```

And now start varnish, this time _with_ SELinux enforcing.

```
[root@cache ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /selinux
Current mode:                   enforcing
Mode from config file:          error (Success)
Policy version:                 24
Policy from config file:        targeted
[root@cache ~]# service varnish start
Starting Varnish Cache:                                    [  OK  ]
[root@cache ~]# ps ax | grep varnish
 2319 ?        SLs    0:00 /usr/sbin/varnishd -P /var/run/varnish.pid -a :80 -f /etc/varnish/default.vcl -T 127.0.0.1:6082 -t 120 -u varnish -g varnish -S /etc/varnish/secret -s malloc,70M
 2321 ?        Sl     0:00 /usr/sbin/varnishd -P /var/run/varnish.pid -a :80 -f /etc/varnish/default.vcl -T 127.0.0.1:6082 -t 120 -u varnish -g varnish -S /etc/varnish/secret -s malloc,70M
```

The module was installed in ```/etc/selinux/targeted/modules/active/modules
```.

```bash
[root@cache modules]# pwd
/etc/selinux/targeted/modules/active/modules
[root@cache modules]# ls varnish*
varnishd.pp  varnishpol.pp
```

It's alongside the existing ```varnishd.pp``` module, which is also found bzipped in:

```bash
[root@cache modules]# locate varnishd.pp
/etc/selinux/targeted/modules/active/modules/varnishd.pp
/usr/share/selinux/targeted/varnishd.pp.bz2
```

So the bzipped file is from the RPM, but what is in ```/etc/selinux``` is more like a configuration file, which makes sense.

```bash
[root@cache modules]# rpm -qf /usr/share/selinux/targeted/varnishd.pp.bz2
selinux-policy-targeted-3.7.19-231.el6_5.3.noarch
[root@cache modules]# rpm -qf varnishd.pp
file /etc/selinux/targeted/modules/active/modules/varnishd.pp is not owned by any package
```

I believe that once that file is in ```/etc/selinux...``` it will be installed on a reboot.

###Conclusion

In the end, by granting varnishd some additional capabilities via SELinux it is allowed to start. However, I still have several questions that I will look to answer in future blog posts:

1. How do the pp files end up getting unzipped and landed in ```/etc/selinux...```?
1. Are the additional capabilities we gave varnishd correct? Is it too much?
1. What's the proper way to add the selinux module? RPM it and install it in the ```/usr/share/selinux``` dir?
1. Should I grab the pp file from Fedora 19 (or later) and use that?

So, still lots of questions, but so far a working varnishd with SELinux still enabled.
