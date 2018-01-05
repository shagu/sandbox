# Sony Xperia Z1 Compact (amami)
Build instructions to compile LineageOS from scratch, using a reproducable lxc container setup. This guide is dedicated for the Xperia Z1 Compact device, which is also known as `amami` or `d5503`. It will use `ubuntu` as an LXC container, so your host system does not matter as long as it has `lxc` installed properly. Ubuntu is the choice here, because it's the preferred distribution in the Sony Xperia AOSP build instructions and also in the official AOSP documentation.

## AOSP
### Setup LXC container
Create an LXC container with ubuntu installed. If xenial is not used by default, pass `-- -r xenial` to the `lxc-create` command.

    lxc-create -n aosp -t ubuntu

Change container config to use the host network

    sed -i "s/lxc.network.type = empty/lxc.network.type = none/g" /var/lib/lxc/aosp/config

Start and attach the new session

    lxc-start  -n aosp
    lxc-attach -n aosp

If the container does not receive any default nameserver, add a default nameserver on every login and re-attach the container.

    echo 'echo "nameserver 8.8.8.8" > /etc/resolv.conf' >> /root/.bashrc && exit
    lxc-attach -n aosp

### Setup Ubuntu
Update all packages and install the required build dependencies.

    apt-get update && apt-get upgrade
    apt-get install openjdk-8-jdk build-essential lzop bc bison g++-multilib git \
        gperf libxml2-utils make zlib1g-dev zip repo vim

### Setup the Build Environment
Add a build user and initialize all git repositories.

    adduser build
    su -l build

    git config --global user.email "build@aosp"
    git config --global user.name "Build User"

    repo init -u https://android.googlesource.com/platform/manifest -b android-7.1.2_r29

Add sony's custom manifest which contains specific data for xperia devices.
    cd .repo
    git clone https://github.com/sonyxperiadev/local_manifests
    cd local_manifests
    git checkout n-mr1_3.10
    cd ~

Grab the Sony AOSP binaries and place them in ~/ of your build user. The binaries can be found on the official sonyxperiadev website: [Download AOSP Binararies for Xperia Devices](https://developer.sonymobile.com/open-devices/list-of-devices-and-resources/)

    unzip SW_binaries_for_Xperia_AOSP_*.zip

Apply the requried changes on the aosp source tree as sony recommends us.

    cd external/toybox
    git fetch https://android.googlesource.com/platform/external/toybox refs/changes/74/265074/1 && git cherry-pick FETCH_HEAD
    git cherry-pick d3e8dd1bf56afc2277960472a46907d419e4b3da
    git cherry-pick 1c028ca33dc059a9d8f18daafcd77b5950268f41
    git cherry-pick cb49c305e3c78179b19d6f174ae73309544292b8

    cd ../../hardware/qcom/audio
    git fetch https://android.googlesource.com/platform/hardware/qcom/audio refs/changes/91/294291/1 && git cherry-pick FETCH_HEAD
    git fetch https://android.googlesource.com/platform/hardware/qcom/audio refs/changes/35/274235/9 && git cherry-pick FETCH_HEAD
    git fetch https://android.googlesource.com/platform/hardware/qcom/audio refs/changes/86/333386/1 && git cherry-pick FETCH_HEAD

    cd ../display
    git revert --no-edit 51b4299f42c61d3a919c8e86c38a85f40902226b
    git revert --no-edit b7d1a389b00370fc9d2a7db1268ce26271ead7e2
    git revert --no-edit f026d04dde743a0524235ae57e2ce8ac5364d44b
    git revert --no-edit 3261eb2236252f9f2510c008fad451411a780b3b
    git fetch https://android.googlesource.com/platform/hardware/qcom/display refs/changes/54/274454/1 && git cherry-pick FETCH_HEAD
    git fetch https://android.googlesource.com/platform/hardware/qcom/display refs/changes/55/274455/1 && git cherry-pick FETCH_HEAD

    cd ../keymaster
    git revert --no-edit 583ecf5ed2a4be0d05229b8c6726680c3836be8b
    git fetch https://android.googlesource.com/platform/hardware/qcom/keymaster refs/changes/70/212570/5 && git cherry-pick FETCH_HEAD
    git fetch https://android.googlesource.com/platform/hardware/qcom/keymaster refs/changes/80/212580/2 && git cherry-pick FETCH_HEAD

    cd ../../../system/core
    git fetch https://android.googlesource.com/platform/system/core refs/changes/52/269652/1 && git cherry-pick FETCH_HEAD
    git fetch https://android.googlesource.com/platform/system/core refs/changes/12/373812/1 && git cherry-pick FETCH_HEAD

    cd ../../packages/apps/Music
    git cherry-pick 6036ce6127022880a3d9c99bd15db4c968f3e6a3

    cd ../../../frameworks/av
    git fetch https://android.googlesource.com/platform/frameworks/av refs/changes/69/343069/1 && git cherry-pick FETCH_HEAD
    git fetch https://android.googlesource.com/platform/frameworks/av refs/changes/70/343070/1 && git cherry-pick FETCH_HEAD
    git fetch https://android.googlesource.com/platform/frameworks/av refs/changes/71/343071/1 && git cherry-pick FETCH_HEAD
    cd ../../

Add proprietary blobs to get the modem working (gsm/umts/lte/4g)

    git clone "https://github.com/SonyAosp/vendor_qcom_firmware" vendor/qcom/firmware

### Download
This will download up to 50GB of source code. Have a break ;)

    repo sync

In case you already messed something up, you might want to use the forced mode instead.

    repo sync --force-sync

### Optional - Add/Remove Apps
This is an example on how to include LineageOS apps or remove any apps during the build process.

    cd packages/apps
    git clone https://github.com/LineageOS/android_packages_apps_Browser.git Browser
    git clone https://github.com/LineageOS/android_packages_apps_CMFileManager.git CMFileManager
    cd

    cd external
    git clone https://github.com/LineageOS/android_external_libtruezip.git libtruezip
    git clone https://github.com/LineageOS/android_external_cyanogen_UICommon.git UICommon

Add `Browser`, `CMFileManager` to `PRODUCT_PACKAGES` and/or remove `Browser2`, `Launcher2` and `QuickSearchBar` if you don't want to use them.

    vim build/target/product/core.mk

### Rebuild the Kernel
Sony uses a prebuild Kernel per default, but this kernel won't boot any amami device. We need to remove the prebuilt in order recompile a working one.

    rm -r device/sony/common-kernel

### Compile
Replace `-j4` with the number of the CPU Cores you have.

    . build/envsetup.sh
    lunch
    make -j4 otapackage

### Flash
In case the update.zip won't work due to a broken recovery image, just flash the raw data directly via fastboot.

    fastboot –S 256M flash boot out/target/product/<device>/boot.img
    fastboot –S 256M flash system out/target/product/<device>/system.img
    fastboot –S 256M flash userdata out/target/product/<device>/userdata.img
    fastboot –S 256M flash recovery out/target/product/<device>/recovery.img

## AICP
### Setup LXC container
Create an LXC container with ubuntu installed. If xenial is not used by default, pass `-- -r xenial` to the `lxc-create` command.

    lxc-create -n aicp -t ubuntu

Change container config to use the host network

    sed -i "s/lxc.network.type = empty/lxc.network.type = none/g" /var/lib/lxc/aicp/config

Start and attach the new session

    lxc-start  -n aicp
    lxc-attach -n aicp

If the container does not receive any default nameserver, add a default nameserver on every login and re-attach the container.

    echo 'echo "nameserver 8.8.8.8" > /etc/resolv.conf' >> /root/.bashrc && exit
    lxc-attach -n aicp

### Setup Ubuntu
Update all packages and install the required build dependencies.

    apt-get update && apt-get upgrade
    apt-get install bc bison bsdmainutils build-essential curl flex gcc-multilib \
        git g++-multilib gnupg gperf imagemagick lib32ncurses5-dev lib32readline6-dev \
        lib32z1-dev libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev \
        libwxgtk3.0-dev libxml2 libxml2-utils lzop make openjdk-8-jdk pngcrush repo \
        rsync schedtool squashfs-tools xsltproc zip zlib1g-dev

### Setup the Build Environment
Add a build user and initialize all git repositories.

    adduser build
    su -l build

    git config --global user.email "build@aicp"
    git config --global user.name "Build User"

    repo init -u https://github.com/AICP/platform_manifest.git -b n7.1
    repo sync

### Compile

    . build/envsetup.sh
    brunch

## LineageOS
## Setup LXC container
Create an LXC container with ubuntu installed. If xenial is not used by default, pass `-- -r xenial` to the `lxc-create` command.

    lxc-create -n lineage -t ubuntu

Change container config to use the host network

    sed -i "s/lxc.network.type = empty/lxc.network.type = none/g" /var/lib/lxc/lineage/config

Start and attach the new session

    lxc-start  -n lineage
    lxc-attach -n lineage

If the container does not receive any default nameserver, add a default nameserver on every login and re-attach the container.

    echo 'echo "nameserver 8.8.8.8" > /etc/resolv.conf' >> /root/.bashrc && exit
    lxc-attach -n lineage

## Setup Ubuntu
Update all packages and install the required build dependencies.

    apt-get update && apt-get upgrade
    apt-get install bc bison bsdmainutils build-essential curl flex gcc-multilib \
        git g++-multilib gnupg gperf imagemagick lib32ncurses5-dev lib32readline6-dev \
        lib32z1-dev libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev \
        libwxgtk3.0-dev libxml2 libxml2-utils lzop make openjdk-8-jdk pngcrush repo \
        rsync schedtool squashfs-tools xsltproc zip zlib1g-dev

## Setup the Build Environment
Add a build user and initialize all git repositories.

    adduser build
    su -l build

    git config --global user.email "build@lineage"
    git config --global user.name "Build User"

    repo init -u https://github.com/LineageOS/android.git -b cm-14.1

    mkdir .repo/local_manifests/
    cat > .repo/local_manifests/roomservice.xml < EOF
    <?xml version="1.0" encoding="UTF-8"?>
    <manifest>
      <!-- replace mainline components by vendor forks -->
      <remove-project name="LineageOS/android_hardware_qcom_audio" />
      <remove-project name="LineageOS/android_hardware_qcom_display" />
      <remove-project name="LineageOS/android_hardware_qcom_fm" />
      <remove-project name="LineageOS/android_hardware_qcom_media" />
      <remove-project name="LineageOS/android_packages_apps_FMRadio" />

      <!-- imports from Sony LineageOS Project -->
      <project name="SonyLOS/android_hardware_qcom_audio" path="hardware/qcom/audio/default" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_hardware_qcom_display" path="hardware/qcom/display" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_hardware_qcom_media" path="hardware/qcom/media" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_device_sony_common-aosp" path="device/sony/common-aosp" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_device_sony_kitakami-common" path="device/sony/kitakami-common" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_device_sony_satsuki" path="device/sony/satsuki" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_device_sony_satsuki_dsds" path="device/sony/satsuki_dsds" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_device_sony_rhine-common" path="device/sony/rhine-common" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_device_sony_honami" path="device/sony/honami" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_device_sony_shinano-common" path="device/sony/shinano-common" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_device_sony_z3" path="device/sony/z3" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_kernel_sony_msm" path="kernel/sony/msm" remote="github" revision="cm-14.1" />
      <project name="SonyLOS/android_hardware_sony_camera" path="hardware/sony/camera" remote="github" revision="cm-14.1" />

      <!-- imports from LineageOS Project -->
      <project name="LineageOS/android_device_sony_common" path="device/sony/common" remote="github" revision="cm-14.1" />
      <project name="LineageOS/android_hardware_sony_thermanager" path="hardware/sony/thermanager" remote="github" revision="cm-14.1" />
      <project name="LineageOS/android_hardware_sony_timekeep" path="hardware/sony/timekeep" remote="github" revision="cm-14.1" />
      <project name="LineageOS/android_hardware_sony_macaddrsetup" path="hardware/sony/macaddrsetup" remote="github" revision="cm-14.1" />

      <!-- imports from official Sony Xperia GitHub -->
      <project name="sonyxperiadev/packages-apps-FMRadio" path="packages/apps/FMRadio" remote="github" revision="master" />
      <project name="sonyxperiadev/device-sony-sepolicy" path="device/sony/sepolicy" remote="github" revision="n-mr0" />
      <project name="sonyxperiadev/vendor-qcom-opensource-fm" path="hardware/qcom/fm" remote="github" revision="master" />
      <project name="sonyxperiadev/vendor-qcom-opensource-dataservices" path="hardware/sony/dataservices" remote="github" revision="master" />

      <!-- add proprietary modem blobs -->
      <project name="SonyAosp/vendor_qcom_firmware" path="vendor/qcom/firmware" remote="github" revision="android-7.0" />

      <!-- add the LOSified fork of sonyxperiadev/device_sony_amami -->
      <project name="derfelot/android_device_sony_amami_los14.1" path="device/sony/amami" remote="github" revision="cm-14.1" />
    </manifest>
    EOF

Grab the Sony AOSP binaries and place them in ~/ of your build user. The binaries can be found on the official sonyxperiadev website: [Download AOSP Binararies for Xperia Devices](https://developer.sonymobile.com/open-devices/list-of-devices-and-resources/)

    unzip SW_binaries_for_Xperia_AOSP_*.zip

## Download
This will download up to 50GB of source code. Have a break ;)

    repo sync

In case you already messed something up, you might want to use the forced mode instead.

    repo sync --force-sync

## Compile
Replace `-j4` with the number of the CPU Cores you have and `12G` with the amount of RAM you would like to use during the build. Set `-M 50G` to the amount of storage that ccache is allowed to use.

    export USE_CCACHE=1
    export ANDROID_JACK_VM_ARGS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx12G"
    prebuilts/misc/linux-x86/ccache/ccache -M 50G

    . build/envsetup.sh
    lunch lineage_amami-userdebug
    make bacon -j4

## Troubleshooting
If you encounter any `ninja_wrapper` errors, disable ninja and try to build again.

    export USE_NINJA=false

