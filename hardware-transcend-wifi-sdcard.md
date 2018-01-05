Transcend Wifi-SD Card 16GB
==========================

* Processor       : ARM926EJ-S rev 5 (v5l)
* BogoMIPS        : 421.06
* Features        : swp half fastmult edsp java
* CPU implementer : 0x41
* CPU architecture: 5TEJ
* CPU variant     : 0x0
* CPU part        : 0x926
* CPU revision    : 5

* Hardware        : KeyASIC Ka2000 EVM
* Revision        : 0000
* Serial          : 0000000000000000

### Software

    BusyBox v1.18.5 (2014-04-23 16:20:41 CST) multi-call binary.
    Linux (none) 2.6.32.28 #137 PREEMPT Fri Mar 22 18:21:52 CST 2013 armv5tejl GNU/Linux

### Links

* documentation: [haxit.blogspot.de](http://haxit.blogspot.de/2013/08/hacking-transcend-wifi-sd-cards.html)
* busybox: [Busybox](http://busybox.net/downloads/binaries/latest/busybox-armv5l)

### Prepare
Place busybox and autorun.sh in the Root dir of the sdcard.

autorun.sh:
    cp /mnt/sd/busybox-armv5l /bin/busybox2
    chmod a+x /bin/busybox2

    /bin/busybox2 telnetd -l /bin/bash &

*note: if you replace the original busybox with your new version, many parts of the webfrontend won't work anymore*

### Boot
Reboot the SD Card, connect to its wifi and type on the host:

    busybox telnet 192.168.11.254

you will see the following:

    eric@surtur ~ $ busybox telnet 192.168.11.254

    Entering character mode
    Escape character is '^]'.


    # busybox2 id
    uid=0(root) gid=0
    #

That's it, root-shell access to your sdcard computer.
