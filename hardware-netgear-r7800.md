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

## LAN-Only WiFi

    export PASS="CHANGEIT"
    export NAME="LAN_ONLY_WIFI"
    export DEVICE="radio1"

    uci set network.$NAME=interface
    uci set network.$NAME.type='bridge'
    uci set network.$NAME.proto='static'
    uci set network.$NAME.ipaddr='10.4.5.1'
    uci set network.$NAME.netmask='255.255.255.0'

    uci set dhcp.$NAME=dhcp
    uci set dhcp.$NAME.start='100'
    uci set dhcp.$NAME.leasetime='4h'
    uci set dhcp.$NAME.limit='250'
    uci set dhcp.$NAME.interface="$NAME"

    uci add firewall zone
    uci set firewall.@zone[-1].name="$NAME"
    uci set firewall.@zone[-1].input='ACCEPT'
    uci set firewall.@zone[-1].output='ACCEPT'
    uci set firewall.@zone[-1].forward='ACCEPT'
    uci set firewall.@zone[-1].network="$NAME"

    uci add firewall forwarding
    uci set firewall.@forwarding[-1].dest='lan'
    uci set firewall.@forwarding[-1].src="$NAME"

    uci add firewall forwarding
    uci set firewall.@forwarding[-1].dest="$NAME"
    uci set firewall.@forwarding[-1].src='lan'

    uci add wireless wifi-iface
    uci set wireless.@wifi-iface[-1].ssid="$NAME"
    uci set wireless.@wifi-iface[-1].device="$DEVICE"
    uci set wireless.@wifi-iface[-1].mode='ap'
    uci set wireless.@wifi-iface[-1].disabled='1'
    uci set wireless.@wifi-iface[-1].encryption='psk2'
    uci set wireless.@wifi-iface[-1].key="$PASS"
    uci set wireless.@wifi-iface[-1].network="$NAME"

    uci commit

## WAN-Only WiFi

    export PASS="CHANGEIT"
    export NAME="WAN_ONLY_WIFI"
    export DEVICE="radio1"

    uci set network.wan.type='bridge'
    uci set network.wan6.type='bridge'

    uci add wireless wifi-iface
    uci set wireless.@wifi-iface[-1].ssid="$NAME"
    uci set wireless.@wifi-iface[-1].device="$DEVICE"
    uci set wireless.@wifi-iface[-1].mode='ap'
    uci set wireless.@wifi-iface[-1].disabled='1'
    uci set wireless.@wifi-iface[-1].encryption='psk2'
    uci set wireless.@wifi-iface[-1].key="$PASS"
    uci set wireless.@wifi-iface[-1].network='wan wan6'
    uci set wireless.@wifi-iface[-1].isolate='1'

    uci commit


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
