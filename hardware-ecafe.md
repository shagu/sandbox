Hercules eCafe
==============

* Processor: ARMv7 800 Mhz
* Hardware: Hercules MX51 eCAFE
* Board: IMX51
* Resolution: 1024x600
* Graphic: IMX51
* Backlight: 255 Steps
* Storage:
    - 8 GB iNAND (eMMC)
    - 1 internal MMC Card Slot
    - 1 external MMC Card Slot

### Sources

* [Toolchain/Kernel(v4C)](http://package.ecafe.hercules.com/Sources/)
* [Restore Image](http://ecafe.hercules.com/us/download-and-services/)

### Kernel

    export ARCH=arm 
    export CROSS_COMPILE=/opt/freescale/usr/local/gcc-4.4.4-glibc-2.11.1-mulitlib-1.0/arm-fsl-Linux-gnueeabi/bin//arm-fsl-Linux-gnueabi-
    export INSTALL_MOD_PATH=./
    make ecafe_defconfig
    make uImage
    make modules
    make modules_install

NAND Install: 

    dd if=uImage of=/dev/mmcblk0 bs=1M seek=1

### Linux

* [Gentoo](http://distfiles.gentoo.org/releases/arm/autobuilds/)
* [Archlinux](http://archlinuxarm.org/os/omap/)
* Debian: Use `debootstrap-cross`

### WiFi

copy /etc/Wireless folder from the ubuntu-tarball to rootfs

### Udev

New udev/systemd requires new kernel features which aren't present in hercules' outdated kernel: 
[Patch](https://github.com/archlinuxarm/PKGBUILDs/blob/01d2c127d2501088cf4f1b971392e6561e2ab11f/core/udev-oxnas/pre-accept4-kernel.patch)

### mplayer

    mplayer -vo sdl -autosync 30 -framedrop -cache 8192

### Video acceleration (on archlinux)

    pacman -S xf86-video-imx
    chmod 777 /dev/gsl_kmod

* obtain [Libraries](http://packages.efikamx.info/pool/main/i/imx-graphics/libkgsl-imx_1.0.3-20110310_armel.deb)
* unpack and move `libkgsl.so.1.0` to `/usr/lib`
* create symlinks

    cd /usr/lib
    ln -s libkgsl.so.1.0 libkgsl.so.1
    ln -s libkgsl.so.1.0 libkgsl.so

