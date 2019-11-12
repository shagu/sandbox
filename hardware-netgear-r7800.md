# OpenWRT on Netgear Nighthawk R7800

## Compile

    git clone https://www.github.com/openwrt/openwrt
    cd openwrt
    ./scripts/feeds update -a
    ./scripts/feeds install -a
    make menuconfig

    # Target System -> `Qualcomm Atheros IPQ806X`
    # Target Profile -> `Netgear Nighthawk X4S R7800`
    # LuCI -> Collections -> `luci`, `luci-ssl`

    make -j16
    du -hs bin/targets/ipq806x/generic/*

## Flash via TFTP

*(Using `extra/tftp-hpa` on Archlinux)*

1. Turn off the power, push and hold the reset button with a pin
2. Turn on the power and wait till power led starts flashing white (after it first flashes orange for a while)
3. Release the pin and tftp the factory img in binary mode. The power led will stop flashing if you succeeded in transferring the image, and the router reboots rather quickly with the new firmware.

*tftp:*
    tftp 192.168.1.1
    mode binary
    put *-factory.img
    quit

## Atheros WDS (AP+Repeater)

*based on: [OpenWRT Atheros WDS](https://openwrt.org/docs/guide-user/network/wifi/atheroswds)*

### Router (Access Point)

- Open `/etc/config/wireless` and add `option wds '1'` to each `config wifi-iface` entry.

### Repeater (STAtion)

- Open `/etc/config/dhcp` and disable DHCP:
  - Add `option ignore '1'` to the `config dhcp 'lan'` section
  - Replace `option dhcpv6 'server'` by `option dhcpv6 'disabled'`

- Open `/etc/config/network` and set IP Address:
  - Set IP-Address: `option ipaddr '10.4.4.2'` *(Repeater IP)*
  - Set Gateway: `option gateway '10.4.4.1'` *(Router IP)*
  - Enable STP: `option stp '1'`

- Open `/etc/config/wireless` and setup WiFi's:
  - Duplicate all Router WiFi entries **twice**
  - On the first, replace `option mode 'ap'` by `option mode 'sta'`
  - On the second, remove `option wds '1'`

- Open `/etc/config/dhcp` and setup DHCP forwarding:
  - Add `list server '10.4.4.1'` to the `config dnsmasq` section *(Router IP)*

- Make sure the devices are enabled
  - Remove `option disabled '1'` from `config wifi-device` section


## Menuconfig for NAS Usage

**Base**
- Target System -> `Qualcomm Atheros IPQ806X`
- Target Profile -> `Netgear Nighthawk X4S R7800`
- LuCI -> Collections -> `luci`, `luci-ssl`
- LuCI -> Applications -> `luci-app-minidlna`, `luci-app-radicale2`, `luci-app-samba4`

**Kernel**
- Kernel Modules -> Cryptographic API -> `kmod-crypto-sha256`, `kmod-crypto-sha512`, `kmod-crypto-xts`, `kmod-crypto-rng`
- Kernel Modules -> Filesystems -> `kmod-fs-cifs`, `kmod-fs-ext4`, `kmod-fs-vfat`, `kmod-fs-ntfs`
- Kernel Modules -> Sound Support -> `kmod-usb-audio`
- Kernel Modules -> USB Support -> `kmod-usb-storage-extras`
- Kernel Modules -> Block Devices -> `kmod-dm`, `kmod-md-mod`, `kmod-md-raid1`, `kmod-loop`

**Utils**
- Utilities -> Encryption -> `cryptsetup`
- Utilities -> Editors -> `vim-full`
- Utilities -> Disc -> `cfdisk`, `mdadm`
- Utilities -> Filesystem -> `dosfstools`, `e2fsprogs`, `ncdu`
- Utilities -> Shells -> `bash`
- Utilities -> Terminal -> `screen`
- Multimedia -> `youtube-dl`
- Sound -> `forked-daapd`, `shairport-sync-mini`
