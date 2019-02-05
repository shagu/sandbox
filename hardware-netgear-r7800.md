# OpenWRT on Netgear Nighthawk R7800

## Compile 
Make all packages available

    ./scripts/feeds update -a
    ./scripts/feeds install -a
    make menuconfig

Target System -> Qualcomm Atheros IPQ806X
Target Profile -> Netgear Nighthawk X4S R7800
LuCI -> Collections -> luci and luci-ssl
LuCI -> Applications -> luci-app-minidlna, luci-app-radicale2, luci-app-samba4

Kernel Modules -> Cryptographic API -> kmod-crypto-sha256, kmod-crypto-sha512, kmod-crypto-xts, kmod-crypto-rng
Kernel Modules -> Filesystems -> kmod-fs-cifs, kmod-fs-ext4, kmod-fs-vfat, kmod-fs-ntfs
Kernel Modules -> Sound Support -> kmod-usb-audio
Kernel Modules -> USB Support -> kmod-usb-storage-extras
Kernel Modules -> Block Devices -> kmod-dm, kmod-md-mod, kmod-md-raid1, kmod-loop

Utilities -> Encryption -> cryptsetup
Utilities -> Editors -> vim-full
Utilities -> Disc -> cfdisk, mdadm
Utilities -> Filesystem -> dosfstools, e2fsprogs, ncdu
Utilities -> Shells -> bash
Utilities -> Terminal -> screen
Multimedia -> youtube-dl
Sound -> forked-daapd, shairport-sync-mini

    make -j16
    du -hs bin/targets/ipq806x/generic/*

## TFTP
1. Turn off the power, push and hold the reset button with a pin
2. Turn on the power and wait till power led starts flashing white (after it first flashes orange for a while)
3. Release the pin and tftp the factory img in binary mode. The power led will stop flashing if you succeeded in transferring the image, and the router reboots rather quickly with the new firmware.

    tftp 192.168.1.1
    mode binary
    put *-factory.img
    quit

## mdadm

    mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1
    mdadm --detail --scan
    vim /etc/config/mdadm
    service mdadm enable
    # mdadm --assemble /dev/md0 /dev/sda1 /dev/sdb1

