GGMM E2-100 Speaker
===================

## About
A cheap AirPlay compatible WiFi speaker. The device is running a kernel in version 2.6.36 and is using a buildroot based userland. The root user mapped to "admin" and the device is using a read-only rootfs. More details below:

```
# cat /proc/cpuinfo
system type             : MT7628
processor               : 0
cpu model               : MIPS 24Kc V5.5
BogoMIPS                : 386.04
wait instruction        : yes
microsecond timers      : yes
tlb_entries             : 32
extra interrupt vector  : yes
hardware watchpoint     : yes, count: 4, address/irw mask: [0x0000, 0x0ff8, 0x0ff8, 0x0ff8]
ASEs implemented        : mips16 dsp
shadow register sets    : 1
core                    : 0
VCED exceptions         : not available
VCEI exceptions         : not available
```

```
# cat passwd
admin:L4lwAM7zGEySM:0:0:Adminstrator:/:/bin/sh
```

```
# mount
rootfs on / type rootfs (rw)
/dev/root on / type squashfs (ro,relatime)
proc on /proc type proc (rw,relatime)
none on /var type ramfs (rw,relatime)
none on /etc type ramfs (rw,relatime)
none on /tmp type ramfs (rw,relatime)
none on /media type ramfs (rw,relatime)
none on /sys type sysfs (rw,relatime)
none on /dev/pts type devpts (rw,relatime,mode=600)
mdev on /dev type ramfs (rw,relatime)
devpts on /dev/pts type devpts (rw,relatime,mode=600)
/dev/mtdblock8 on /mnt type jffs2 (rw,relatime)
/dev/mtdblock9 on /vendor type jffs2 (rw,relatime)
/dev/mtdblock9 on /tmp/web type jffs2 (rw,relatime)
```

```
# df
Filesystem           1k-blocks      Used Available Use% Mounted on
rootfs                    5632      5632         0 100% /
/dev/root                 5632      5632         0 100% /
/dev/mtdblock8             512       196       316  38% /mnt
/dev/mtdblock9            6144       732      5412  12% /vendor
/dev/mtdblock9            6144       732      5412  12% /tmp/web
```

It seems like the device is using the ralink NVRAM as its main storage for user values. Several scripts were found to reset those values inside and replacing some of its default. Most of the scripts are using the `nvram_get` utility in order to obtain a specific entry of the NVRAM. The command `ralink_init show 2860` basically shows every entry of the ralink 2860 chip, where `ralink_init clear 2860` will clear all of its content. In order to get a single value, `nvram_get SSID1` can be used or `nvram_set SSID1 "bathroom"` to set a value.

## Motivation
It turned out, whenever I tried to set the devicename to "bathroom", it started to append a random string to the end of the name, so it became `bathroomssConnectEm` and something along those lines. Frustrated by that, I started playing around with the devicename. It turned out very quickly, it doesn't strip anything I write into the field, I was able to use whitespaces, linebreaks and all that.

- Problem: Device name gets overwritten/appended by other strings
- Observation: Device name accepts all inputs and looks wonky
- Result: Obvious...

## Remote Shell
By default, the speaker comes in AP mode:

    SSID: GGMM_E2_XXX
    Password: ggmm123456

The device is running a dnsmasq service that will provide you a `10.10.10.x` IP address,
the speaker itself can be found at 10.10.10.254. On port 80 you will be greeted by a webfrontend.
Since the webfrontend is provided by cgi-scripts running on a lighttpd, it's clear to play around with some parsing issues first.
Fortunately the only input field you have - the device name - is exploitable.

Knowing this, obtaining a remote shell is as simple as setting the hostname to:

    abc`telnetd -l /bin/sh`

It's required to either reboot the device or putting it into another WiFi to get our command executed. Since telnetd is already enabled by default in the shipped busybox version, you'll be greeted by a beautiful telnet shell when connecting.
The device uses `admin` as the default root user, the password is `admin` aswell.
