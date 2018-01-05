pandaboard
==========

* About: 
  [wikipedia](https://de.wikipedia.org/wiki/PandaBoard),
  [eLinux](http://elinux.org/PandaBoard),
  [Gentoo Guide](http://dev.gentoo.org/~armin76/arm/pandaboard/install.xml)
* Processor: Cortex-A9 DualCore Processor @1Ghz.
* Layout: OMAP4430-SoC
* RAM: 1GB

### Toolchain
    crossdev -S armv7a-hardfloat-linux-gnueabi

### UBoot
    wget ftp://ftp.denx.de/pub/u-boot/u-boot-latest.tar.bz2 
    tar xjpf u-boot-latest.tar.bz2 && cd u-boot-*
    make ARCH=arm CROSS_COMPILE=armv7a-hardfloat-linux-gnueabi- omap4_panda_config
    make ARCH=arm CROSS_COMPILE=armv7a-hardfloat-linux-gnueabi-

### Kernel
    make ARCH=arm CROSS_COMPILE=armv7a-hardfloat-linux-gnueabi- omap2plus_defconfig
    make ARCH=arm CROSS_COMPILE=armv7a-hardfloat-linux-gnueabi- uImage

### Partitioning

* mmcblk0p1: [type: vfat] [flags: boot]
* mmcblk0p2: [type: ext4] [flags: -]

### Flashing
    cp u-boot-*/MLO /mnt/mmcblk0p1
    cp u-boot-*/u-boot.img /mnt/mmcblk0p1
    cp kernel/arch/arm/boot/uImage /mnt/mmcblk0p1
