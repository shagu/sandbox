Ubuntu inside FreeBSD Jail
==========================

### Prepare for linux compatible jails

    kldload fdescfs linprocfs linsysfs tmpfs

    echo 'linux_enable="YES"' >> /etc/rc.conf
    echo 'compat.linux.osrelease=2.6.18' >> /etc/sysctl.conf
    sysctl compat.linux.osrelease=2.6.32

    echo 'jail_enable="YES"' >> /etc/rc.conf
    echo 'jail_conf="/home/eric/jails/etc/jail.conf"' >> /etc/rc.conf
    echo 'jail_list="ubuntu"' >> /etc/rc.conf

### Setup the jail

    mkdir -p /home/eric/jails/etc

    cat > /home/eric/jails/etc/jail.conf << EOF
    ubuntu {
	    path = /home/eric/jails/ubuntu;
	    allow.mount;
	    mount.devfs;
	    host.hostname = ubuntu;
	    mount.fstab="/home/eric/jails/etc/fstab.ubuntu";
	    ip4.addr = 127.0.0.10;
	    interface = lo0;
	    exec.start = "/etc/init.d/rc 3";
	    exec.stop = "/etc/init.d/rc 0";
    }
    EOF

    cat > /home/eric/jails/etc/fstab.ubuntu << EOF 
    linsys   /home/eric/jails/ubuntu/sys         linsysfs  rw          0 0
    linproc  /home/eric/jails/ubuntu/proc        linprocfs rw          0 0
    #tmpfs    /home/eric/jails/ubuntu/lib/init/rw tmpfs     rw,mode=777 0 0
    EOF


### Install Ubuntu

    debootstrap --no-check-gpg --arch=i386 trusty /home/eric/jails/ubuntu http://archive.ubuntu.com/ubuntu/

    umount /home/eric/jails/ubuntu/proc
    umount /home/eric/jails/ubuntu/dev
    umount /home/eric/jails/ubuntu/sys
    umount /home/eric/jails/ubuntu/lib/init/rw

    cat > /home/eric/jails/ubuntu/etc/apt/sources.list << EOF
    deb http://de.archive.ubuntu.com/ubuntu trusty main universe multiverse restricted
    EOF

### Launch the jail

    jail -f /home/eric/jails/etc/jail.conf -c ubuntu
    jls
    jexec 1 /bin/bash


### Setup Network

    kldload pf
    echo 'pf_enable="YES"' >> /etc/rc.conf
    echo "nat on wlan0 from 127.0.0.10 to any -> (wlan0)" >> /etc/pf.conf 
