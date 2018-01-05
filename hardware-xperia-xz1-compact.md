Sony Xperia XZ1 Compact (lilac)
===============================

## Lineage OS [TODO]
### Prepare LXC

    lxc-create -n lineage -t ubuntu

    sed -i "s/lxc.net.0.type = empty/lxc.net.0.type = none/g" /var/lib/lxc/lineage/config

    lxc-start  -n lineage
    lxc-attach -n lineage

    echo 'echo "nameserver 8.8.8.8" > /etc/resolv.conf' >> /root/.bashrc && exit
    lxc-attach -n lineage

    apt-get update && apt-get upgrade
    apt-get install bc bison bsdmainutils build-essential curl flex gcc-multilib \
        git g++-multilib gnupg gperf imagemagick lib32ncurses5-dev lib32readline6-dev \
        lib32z1-dev libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev \
        libwxgtk3.0-dev libxml2 libxml2-utils lzop make openjdk-8-jdk pngcrush repo \
        rsync schedtool squashfs-tools xsltproc zip zlib1g-dev

### Setup Build Environment

    su -l ubuntu

    git config --global user.email "build@lineage"
    git config --global user.name "Build User"

    repo init -u git://github.com/LineageOS/android.git -b lineage-15.0
    mkdir .repo/local_manifests
    cat > .repo/local_manifests/roomservice.xml << EOF
    <?xml version="1.0" encoding="UTF-8"?>
    <manifest>
      <!-- SONY -->
      <project name="LineageOS/android_hardware_sony_macaddrsetup" path="hardware/sony/macaddrsetup" remote="github" />
      <project name="LineageOS/android_hardware_sony_thermanager" path="hardware/sony/thermanager" remote="github" />
      <project name="LineageOS/android_hardware_sony_timekeep" path="hardware/sony/timekeep" remote="github" />
      <project name="LineageOS/android_device_qcom_common" path="device/qcom/common" remote="github" />
      <project name="LineageOS/android_device_sony_common" path="device/sony/common" remote="github" />
      <project name="cryptomilk/android_kernel_sony_msm8998" path="kernel/sony/msm8998" remote="github" />
      <project name="cryptomilk/android_device_sony_common-treble" path="device/sony/common-treble" remote="github" />
      <project name="cryptomilk/android_device_sony_yoshino" path="device/sony/yoshino" remote="github" />
      <project name="cryptomilk/android_device_sony_lilac" path="device/sony/lilac" remote="github" />
      <!--<project name="cryptomilk/proprietary_vendor_sony_lilac" path="vendor/sony/lilac" remote="github" />-->
    </manifest>
    EOF

    repo sync

### Build

    source build/envsetup.sh
    lunch lineage_lilac-userdebug

     make -j8 bacon
