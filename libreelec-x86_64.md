Build LibreELEC x86_64
======================

### Setup and prepare LXC Container

    lxc-create -n libreelec -t ubuntu
    sed -i "s/lxc.net.0.type = empty/lxc.net.0.type = none/g" /var/lib/lxc/libreelec/config

    lxc-start  -n libreelec
    lxc-attach -n libreelec

    echo 'echo "nameserver 8.8.8.8" > /etc/resolv.conf' >> /root/.bashrc && exit
    lxc-attach -n libreelec

    source /etc/environment
    source /etc/profile

    apt-get update && apt-get upgrade
    apt-get install build-essential gcc make git unzip wget xz-utils python \
      bc patchutils gawk gperf zip lzop xfonts-utils xfonts-utils xfonts-utils \
      xsltproc default-jre libncurses5-dev libjson-perl libxml-parser-perl libz-dev
    dpkg-reconfigure dash
    # select "no"

### Get the Sources

    su -l ubuntu
    source /etc/environment

    git config --global user.email "build@libreelec"
    git config --global user.name "Build User"

    git clone https://github.com/LibreELEC/LibreELEC.tv.git
    cd ~/LibreELEC.tv

### Patch LibreELEC

This patch allows to build and pre-install addons during the build process:

    diff --git a/config/show_config b/config/show_config
    index 3de77b4..efa2854 100644
    --- a/config/show_config
    +++ b/config/show_config
    @@ -156,6 +156,10 @@ show_config() {
       config_message="$config_message\n - Default Skin:\t\t\t $SKIN_DEFAULT"
       config_message="$config_message\n - Include extra fonts:\t\t\t $KODI_EXTRA_FONTS"

    +  for config_addon in $ADDITIONAL_ADDONS; do
    +    config_message="$config_message\n - Include Addon:\t\t\t $config_addon"
    +  done
    +
       if [ "$(type -t show_distro_config)" = "function" ]; then
         show_distro_config
       fi
    diff --git a/distributions/LibreELEC/options b/distributions/LibreELEC/options
    index 0855ebb..4cbffe2 100644
    --- a/distributions/LibreELEC/options
    +++ b/distributions/LibreELEC/options
    @@ -64,6 +64,11 @@
     # e.g. ADDITIONAL_DRIVERS="DRIVER1 DRIVER2"
       ADDITIONAL_DRIVERS="RTL8192CU RTL8192DU RTL8192EU RTL8188EU RTL8812AU"

    +# additional addons to install:
    +# Space separated list is supported,
    +# e.g. ADDITIONAL_ADDONS="game.libretro game.libretro.2048 vdr-addon"
    +  ADDITIONAL_ADDONS="game.libretro game.libretro.2048 vdr-addon"
    +
     # build and install bluetooth support (yes / no)
       BLUETOOTH_SUPPORT="yes"

    diff --git a/scripts/image b/scripts/image
    index d21e067..3e4211a 100755
    --- a/scripts/image
    +++ b/scripts/image
    @@ -167,6 +167,9 @@ $SCRIPTS/install network
     # Multimedia support
     [ ! "$MEDIACENTER" = "no" ] && $SCRIPTS/install mediacenter

    +# Additional addons
    +[ ! "$MEDIACENTER" = "no" ] && $SCRIPTS/include_addon $ADDITIONAL_ADDONS
    +
     # Sound support
     [ "$ALSA_SUPPORT" = "yes" ] && $SCRIPTS/install alsa

    diff --git a/scripts/include_addon b/scripts/include_addon
    new file mode 100755
    index 0000000..bf64425
    --- /dev/null
    +++ b/scripts/include_addon
    @@ -0,0 +1,18 @@
    +#!/bin/bash
    +
    +for addon in $@; do
    +  echo -e "\e[1;33m  ADDON \e[0m$addon"
    +  . config/options $addon
    +
    +  $SCRIPTS/create_addon $addon &> /dev/null
    +
    +  cp -rf "$ADDON_BUILD/$PKG_ADDON_ID" \
    +         "$INSTALL/usr/share/kodi/addons/"
    +
    +  # enable non-systemd addons
    +  SYSTEMD_SERVICE="$INSTALL/usr/share/kodi/addons/$PKG_ADDON_ID/system.d/$PKG_ADDON_ID.service"
    +  if [ ! -f "$SYSTEMD_SERVICE" ]; then
    +    ADDON_MANIFEST=$INSTALL/usr/share/kodi/system/addon-manifest.xml
    +    xmlstarlet ed -L --subnode "/addons" -t elem -n "addon" -v "$PKG_ADDON_ID" $ADDON_MANIFEST
    +  fi
    +done

### Configure LibreELEC

The main configuration can be done by editing the file: `distributions/LibreELEC/options`

### Start the build

    PROJECT=Generic ARCH=x86_64 make

