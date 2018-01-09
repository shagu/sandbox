Pulse Audio Network Sink
========================

* Hardware: OrangePi Zero
* Distribution: [armbian](https://www.armbian.com/orange-pi-zero/)

### Software

    apt-get update
    apt-get install libasound2 libasound2-plugins alsa-utils alsa-oss \
        pulseaudio pulseaudio-utils avahi-daemon pulseaudio-module-zeroconf

Modify `/boot/armbianEnv.txt` to enable the analog output of the orange-pi:

    overlays=analog-codec

In `/etc/rc.local` add a few lines to setup volumes and autostart pulseaudio:

    /usr/bin/pulseaudio
    /usr/bin/amixer sset "Line Out" 100%
    /usr/bin/amixer sset "DAC" 100%

Modify `/etc/pulse/daemon.conf` to adjust pulseaudio's default settings:

    daemonize = yes
    allow-module-loading = yes
    allow-exit = no
    system-instance = yes

In `/etc/pulse/system.pa` add the required modules:

    load-module module-native-protocol-tcp auth-anonymous=1
    load-module module-zeroconf-discover
    load-module module-zeroconf-publish

Disable IPv6 in `/etc/sysctl.conf` to remove duplicates in your sink-list:

    net.ipv6.conf.all.disable_ipv6=1

Also modify `/etc/avahi/avahi-daemon.conf` for the same reasons

    use-ipv6=no

Set the desired hostname in `/etc/hostname`

### Helpful Commands

    pactl list short sinks

