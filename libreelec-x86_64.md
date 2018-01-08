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

### Start the build

    su -l ubuntu
    source /etc/environment

    git config --global user.email "build@libreelec"
    git config --global user.name "Build User"

    git clone https://github.com/LibreELEC/LibreELEC.tv.git
    cd ~/LibreELEC.tv

    PROJECT=Generic ARCH=x86_64 make
