odroid-x
========

### About

* Processor: Samsung Exynos4412 Cortex-A9 Quad Core 1.4Ghz with 1MB L2 cache
* Memory: 1024MB LP-DDR2
* 3D Accelerator: Mali-400 Quad Core
* LAN: 10/100Mbps Ethernet

### Links

* [Kernel](http://dev.odroid.com/projects/linux)
* [OpenGL ES](http://dev.odroid.com/projects/opengles-linux)
* [Boot Sequence](http://dev.odroid.com/projects/4412boot)
* [uBoot](http://dev.odroid.com/projects/uboot)

### Kernel

source:	`git clone https://github.com/hardkernel/linux.git`
export: `ARCH,CROSS_COMPILE,INSTALL_MOD_PATH`

    make odroidx_ubuntu_defconfig
    make zImage modules modules_install

### Uboot

source: `git clone https://github.com/hardkernel/u-boot.git`
export: `ARCH,CROSS_COMPILE,INSTALL_MOD_PATH`

    make distclean 
    make smdk4412_config
    make

### Bootscript

    mkimage -A arm -T script -C none -n "Boot.scr for odroid-x" -d boot.txt boot.scr

### Flashing

    http://dev.odroid.com/projects/4412boot/wiki/FrontPage?action=download&value=boot.tar.gz 

### Serial Console

    T0:23:respawn:/sbin/getty -L ttySAC0 115200 vt100
