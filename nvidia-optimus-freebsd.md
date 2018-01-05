NVIDIA Optimus on FreeBSD
=========================

### Install Dependencies

    pkg install gcc
    pkg install libjpeg-turbo
    pkg install virtualgl

### Build the driver

    cd /usr/ports/x11/nvidia-drivers
    make clean install

### Setup X11
    cat > /etc/X11/xorg.conf.nv << EOF
    Section "ServerLayout"
        Identifier "Layout0"
        Option "AutoAddDevices" "false"
        Option "AllowEmptyInput" "False"
        InputDevice "fake" "CorePointer"
    EndSection
    Section "Device"
        Identifier "Device1"
        Driver "nvidia"
        VendorName "NVIDIA Corporation"
        BusID "PCI:01:00:0"
        Option "NoLogo" "true"
        Option "UseEDID" "false"
        Option "ConnectedMonitor" "DFP"
    EndSection

    Section "InputDevice"
            Identifier "fake"
            Driver ""
    EndSection
    EOF

### Run Application

    kldload nvidia
    Xorg -config /etc/X11/xorg.conf.nv -sharevts -noreset :8
    /usr/local/VirtualGL/bin/vglrun -ld /usr/local/lib/.nvidia/ -d ":8" winecfg
    kldunload nvidia
