cubox-i4 pro
============

### prepare
build the toolchain:

    crossdev -S armv7a-hardfloat-linux-gnueabi

make sure the first partition has an offset of 1MB otherwise 
the bootloader will destroy your first partition.

    cfdisk /dev/mmcblk0
    mkfs.ext4 /dev/mmcblk0p1
    #mkfs.ext4 -O ^has_journal -E stride=2,stripe-width=1024 -b 4096 /dev/sdX1

### gentoo
source: `http://distfiles.gentoo.org/releases/arm/autobuilds/current-stage3-armv7a_hardfp/`

    tar xfjp stage3-armv7a_hardfp-*.tar.bz2 -C /mnt/mmcblk0p1
    openssl passwd -1
    # add the hash to your /mnt/mmcblk0p1/etc/shadow

### inittab (serial login)

    s0:12345:respawn:/sbin/agetty -L 115200 ttymxc0 vt100

### u-boot

    git clone https://github.com/SolidRun/u-boot-imx6.git
    cd u-boot-imx6
    export ARCH=arm
    export CROSS_COMPILE=/usr/bin/armv7a-hardfloat-linux-gnueabi-
    make mx6_cubox-i_config
    make

files:

    SPL
    u-boot.img

uEnv.txt:

    bootfile=/boot/uImage
    mmcargs=setenv bootargs root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=hdmi consoleblank=0 console=ttymxc0,115200

install:

    dd if=SPL of=/dev/mmcblk0 bs=1K seek=1
    dd if=u-boot.img of=/dev/mmcblk0 bs=1K seek=42
    cp uEnv.txt /mnt/mmcblk0p1

### kernel

    export ARCH=arm
    export CROSS_COMPILE=/usr/bin/armv7a-hardfloat-linux-gnueabi-
    git clone https://github.com/SolidRun/linux-imx6.git
    cd linux-imx6
    make imx6_cubox-i_hummingboard_defconfig
    make uImage
    make modules

install:

    INSTALL_MOD_PATH=/mnt/mmcblk0p1 make modules_install
    mkdir /mnt/mmcblk0p1/boot
    cp arch/arm/boot/uImage /mnt/mmcblk0p1/boot
