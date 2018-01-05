Build Recalbox x86_64 (NVIDIA)
==============================

### Setup and prepare LXC Container

    lxc-create -n recalbox -t ubuntu
    sed -i "s/lxc.net.0.type = empty/lxc.net.0.type = none/g" /var/lib/lxc/recalbox/config

    lxc-start  -n recalbox
    lxc-attach -n recalbox

    echo 'echo "nameserver 8.8.8.8" > /etc/resolv.conf' >> /root/.bashrc && exit
    lxc-attach -n recalbox

    source /etc/environment
    source /etc/profile

    apt-get update && apt-get upgrade
    apt-get install build-essential git libncurses5-dev qt5-default qttools5-dev-tools \
    mercurial libdbus-glib-1-dev texinfo zip openssh-client libxml2-utils \
    software-properties-common wget cpio bc locales rsync imagemagick \
    nano vim automake mtools dosfstools subversion openjdk-8-jdk libssl-dev

### Start the build

    su -l ubuntu
    source /etc/environment

    git config --global user.email "build@recalbox"
    git config --global user.name "Build User"

    git clone https://gitlab.com/recalbox/recalbox.git recalbox
    cd recalbox

    git submodule init
    git submodule update

*If you have any troubles with the submodule, please have a look here: [Issue](https://gitlab.com/recalbox/recalbox/issues/304)*

### Add Custom Config

This adds a systemwide pulseaudio daemon, vdr daemon, nvidia drivers and intel microcode.

    cat >> ./configs/recalbox-x86_64_defconfig << EOF
    # pulseaudio
    BR2_PACKAGE_PULSEAUDIO=y
    BR2_PACKAGE_PULSEAUDIO_DAEMON=y
    # vdr
    BR2_PACKAGE_VDR=y
    BR2_PACKAGE_VDR_PLUGIN_VNSISERVER=y
    # nvidia
    BR2_PACKAGE_MESA3D=n
    BR2_PACKAGE_NVIDIA_DRIVER=y
    BR2_PACKAGE_NVIDIA_DRIVER_XORG=y
    BR2_PACKAGE_NVIDIA_DRIVER_PRIVATE_LIBS=y
    BR2_PACKAGE_NVIDIA_DRIVER_MODULE=y
    # intel microcode
    BR2_PACKAGE_INTEL_MICROCODE=y
    EOF

### Cherry-pick the latest NVIDIA driver

    cd buildroot/package/nvidia-driver/
    git cherry-pick 7ecfd5db6f3593b4d6fefa75f28ebba34f3076c5
    git cherry-pick 4ef04c476c79c7efe05b8befc35eb20997fcaaa4
    git cherry-pick 2068c7c6a810cdaf55240faf15c226ce3b308f1b
    git cherry-pick 47ef5def00f5d991dafe53a6eb6566147124444e
    git cherry-pick 05a86bdf1fa9071de9701fba058d47d80a0925bd
    git cherry-pick e3dab26f2ea77675f888c0c778c31db01c284ef5
    git cherry-pick 985fe64026c2b0ab72877f92636328cfb4433418
    cd -

### Run the build

    make recalbox-x86_64_defconfig
    make
