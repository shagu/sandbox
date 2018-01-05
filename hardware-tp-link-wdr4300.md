TP-Link WDR4300
===============

# OpenWRT
    git clone https://www.github.com/openwrt/openwrt
    cd openwrt
    ./scripts/feeds update packages luci
    ./scripts/feeds install -a -p luci
    make menuconfig

## Configure

* Target System: `Atheros AR7xxx/AR9xxx`
* Subtarget: `Generic`
* Target Profile: `TP-LINK TL-WDR4300 v1`
* Luci -> 1. Collections -> `luci` and `luci-ssl`

## Build

    make
    ls bin/targets/ar71xx/generic/openwrt-ar71xx-generic-tl-wdr4300-v1-squashfs-*
