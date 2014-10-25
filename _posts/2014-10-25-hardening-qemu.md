---
layout:     post
title:      Experimenting with Hardening QEMU
date:       2014-10-25 14:00:00
categories:
  - qemu
  - openstack
  - hardening

---

In this post I will explore compiling QEMU with less option/drivers, thereby removing some code to theoretically make QEMU more secure.

<!-- more -->

First off I want to note that this is an experiment. I'm not working on this for a production system, and I haven't worked a lot with QEMU compilation nor what code is necessary or unnecessary for my particular application. I'm just messing around, trying to learn some things about qemu, what code is in it by default, and what code could be removed via compile options.

Having said that, the [OpenStack Security Guide](http://docs.openstack.org/security-guide/content/ch052_devices.html) does talk about compiling a custom QEMU so that it can be "hardened," which includes turning off options/drivers that aren't needed.

### Environment

I'm going to try this out on Ubuntu Trusty 14.04 using a Vagrant box.

First, we need git.

```bash
root# apt-get install git -y
```

Next clone the QEMU project code.

```bash
root# git clone git://git.qemu-project.org/qemu.git
```

Get all the other things you need to compile QEMU.

```bash
root# apt-get build-dep qemu -y
```

### Configure options

Using ```./configure --help``` we can see there are all kinds of things we can disable and thus probably break QEMU. :)

```bash
root# cd qemu/
root# ./configure --help | grep disable
  --disable-debug-tcg      disable TCG debugging (default)
  --disable-debug-info     disable debugging information
  --disable-sparse         disable sparse checker (default)
  --disable-strip          disable stripping binaries
  --disable-werror         disable compilation abort on warning
  --disable-stack-protector disable compiler-provided stack protection
  --disable-sdl            disable SDL
  --disable-gtk            disable gtk UI
  --disable-virtfs         disable VirtFS
  --disable-vnc            disable VNC
  --disable-cocoa          disable Cocoa (Mac OS X only)
  --disable-xen            disable xen backend driver support
  --disable-xen-pci-passthrough
  --disable-brlapi         disable BrlAPI
  --disable-vnc-tls        disable TLS encryption for VNC server
  --disable-vnc-sasl       disable SASL encryption for VNC server
  --disable-vnc-jpeg       disable JPEG lossy compression for VNC server
  --disable-vnc-png        disable PNG compression for VNC server (default)
  --disable-vnc-ws         disable Websockets support for VNC server
  --disable-curses         disable curses output
  --disable-curl           disable curl connectivity
  --disable-fdt            disable fdt device tree
  --disable-bluez          disable bluez stack connectivity
  --disable-slirp          disable SLIRP userspace network connectivity
  --disable-kvm            disable KVM acceleration support
  --disable-rdma           disable RDMA-based migration support
  --disable-system         disable all system emulation targets
  --disable-user           disable all user emulation targets
  --disable-linux-user     disable all linux usermode emulation targets
  --disable-bsd-user       disable all BSD usermode emulation targets
  --disable-guest-base     disable GUEST_BASE support
  --disable-pie            do not build Position Independent Executables
  --disable-uuid           disable uuid support
  --disable-vde            disable support for vde network
  --disable-netmap         disable support for netmap network
  --disable-linux-aio      disable Linux AIO support
  --disable-cap-ng         disable libcap-ng support
  --disable-attr           disable attr and xattr support
  --disable-blobs          disable installing provided firmware blobs
  --disable-docs           disable documentation build
  --disable-vhost-net      disable vhost-net acceleration support
  --disable-spice          disable spice
  --disable-libiscsi       disable iscsi support
  --disable-libnfs         disable nfs support
  --disable-smartcard-nss  disable smartcard nss support
  --disable-libusb         disable libusb (for usb passthrough)
  --disable-usb-redir      disable usb network redirection support
  --disable-guest-agent    disable building of the QEMU Guest Agent
  --disable-seccomp        disable seccomp support
  --disable-coroutine-pool disable coroutine freelist (worse performance)
  --disable-glusterfs      disable GlusterFS backend
  --disable-archipelago    disable Archipelago backend
  --disable-tpm            disable TPM support
  --disable-libssh2        disable ssh block device support
  --disable-vhdx           disable support for the Microsoft VHDX image format
  --disable-quorum         disable quorum block filter support
  --disable-numa           disable libnuma support
  ```

If I run configure with only the ```--target-list=x86_64-softmmu``` option, it outputs what is turned on and off by default.

```bash
root# ./configure --target-list=x86_64-softmmu
Disabling libtool due to broken toolchain support
Install prefix    /usr/local
BIOS directory    /usr/local/share/QEMU
binary directory  /usr/local/bin
library directory /usr/local/lib
module directory  /usr/local/lib/QEMU
libexec directory /usr/local/libexec
include directory /usr/local/include
config directory  /usr/local/etc
local state directory   /usr/local/var
Manual directory  /usr/local/share/man
ELF interp prefix /usr/gnemul/QEMU-%M
Source path       /home/vagrant/QEMU
C compiler        cc
Host C compiler   cc
C++ compiler      c++
Objective-C compiler cc
ARFLAGS           rv
CFLAGS            -O2 -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -pthread -I/usr/include/glib-2.0 -I/usr/lib/x86_64-linux-gnu/glib-2.0/include   -g
QEMU_CFLAGS       -I/usr/include/pixman-1    -Werror -fPIE -DPIE -m64 -D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -Wstrict-prototypes -Wredundant-decls -Wall -Wundef -Wwrite-strings -Wmissing-prototypes -fno-strict-aliasing -fno-common  -Wendif-labels -Wmissing-include-dirs -Wempty-body -Wnested-externs -Wformat-security -Wformat-y2k -Winit-self -Wignored-qualifiers -Wold-style-declaration -Wold-style-definition -Wtype-limits -fstack-protector-all   -I/usr/include/p11-kit-1   -I/usr/include/p11-kit-1    -I/usr/include/libpng12   -I/usr/include/spice-server -I/usr/include/glib-2.0 -I/usr/lib/x86_64-linux-gnu/glib-2.0/include -I/usr/include/pixman-1 -I/usr/include/spice-1   -I/usr/include/libusb-1.0
LDFLAGS           -Wl,--warn-common -Wl,-z,relro -Wl,-z,now -pie -m64 -g
make              make
install           install
python            python -B
smbd              /usr/sbin/smbd
module support    no
host CPU          x86_64
host big endian   no
target list       x86_64-softmmu
tcg debug enabled no
gprof enabled     no
sparse enabled    no
strip binaries    yes
profiler          no
static build      no
pixman            system
SDL support       yes
GTK support       no
VTE support       no
curses support    yes
curl support      yes
mingw32 support   no
Audio drivers     oss
Block whitelist (rw)
Block whitelist (ro)
VirtFS support    yes
VNC support       yes
VNC TLS support   yes
VNC SASL support  yes
VNC JPEG support  no
VNC PNG support   yes
VNC WS support    yes
xen support       yes
brlapi support    yes
bluez  support    yes
Documentation     yes
GUEST_BASE        yes
PIE               yes
vde support       no
netmap support    no
Linux AIO support yes
ATTR/XATTR support yes
Install blobs     yes
KVM support       yes
RDMA support      no
TCG interpreter   no
fdt support       yes
preadv support    yes
fdatasync         yes
madvise           yes
posix_madvise     yes
sigev_thread_id   yes
uuid support      yes
libcap-ng support yes
vhost-net support yes
vhost-scsi support yes
Trace backends    nop
spice support     yes (0.12.6/0.12.4)
rbd support       yes
xfsctl support    yes
nss used          no
libusb            yes
usb net redir     yes
GLX support       yes
libiscsi support  no
libnfs support    no
build guest agent yes
QGA VSS support   no
seccomp support   yes
coroutine backend ucontext
coroutine pool    yes
GlusterFS support no
Archipelago support no
gcov              gcov
gcov enabled      no
TPM support       yes
libssh2 support   no
TPM passthrough   yes
QOM debugging     yes
vhdx              yes
Quorum            yes
lzo support       no
snappy support    no
NUMA host support no
```

### Disable bluez

I see ```bluez``` there, guessing that is something to do with bluetooth (apparently it is the official [bluetooth linux stack](http://www.bluez.org/)). Trying not to make a joke about QEMU having the "bluez." Why would we need bluetooth in QEMU? There is probably a good reason, but let's pick on it anyways and disable it.

```bash
root# ./configure --target-list=x86_64-softmmu  --disable-bluez
```

There, now it's configured not to compile in bluez support.

```bash
root# ./configure --target-list=x86_64-softmmu  --disable-bluez | grep bluez
bluez  support    no
```

### Make install

If I run ```make install``` QEMU will be installed to the default locations.

```bash
root# /usr/local/bin/qemu-system-x86_64 -version
QEMU emulator version 2.1.50, Copyright (c) 2003-2008 Fabrice Bellard
```

If you don't want to install it, the QEMU binary is in:

```bash
root# ./x86_64-softmmu/qemu-system-x86_64 -version
QEMU emulator version 2.1.50, Copyright (c) 2003-2008 Fabrice Bellard
```

### Conclusion

We don't always have to run a stock system. In some cases it may be completely valid to work towards enhancing security by reducing the attack surface of the QEMU system through disabling drivers and options. In fact the OpenStack security guide mentions doing just that, as well as other compile time options to harden QEMU. Of course knowing what to leave in and what to take out, as well as being able to support that decision in production, is where all the real work is.

In future posts I hope to continue to explore reducing the attack surface of QEMU.
