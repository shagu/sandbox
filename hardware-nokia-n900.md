nokia n900
==========

* Processor: TI Omap3 ARMv7 Processor rev 3 (v7l) 500 Mhz
* Hardware: Nokia RX-51 board
* Board Layout: omap3
* Default Kernel: 2.6.28.10

### Sources

* [Flasher](http://tablets-dev.nokia.com/maemo-dev-env-downloads.php)
* [Firmware](http://tablets-dev.nokia.com/nokia_N900.php)

### Flashing

#### eMMC image:

    /path/to/flasher -F <emmc-image>.bin -f

hold down the "u" key on the phone, and connect it to the PC

#### Firmware image:

    /path/to/flasher -F <firmware-image>.bin -f -R

hold down the "u" key on your phone, and connect it to your PC

### Add `extras`-repository

create a new entry:

    catalog name: extras
    web address: http://repository.maemo.org/extras
    distribution: (empty)
    components: free non-free

### Root

    package: gainroot
    repository: extra

### Tethering

    package:    mobile-hotspot
    repository: extra
