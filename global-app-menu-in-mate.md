Global App-Menu in MATE
=======================

### Install dependencies

    pacman -S cmake appmenu-qt4 libdbusmenu-glib libdbusmenu-gtk2 libdbusmenu-gtk3

### Build the source

    git clone https://github.com/rilian-la-te/vala-panel-appmenu.git
    cd vala-panel-appmenu
    git submodule init && git submodule update
    mkdir build
    cd build

    cmake -DENABLE_MATE=ON -DENABLE_UNITY_GTK_MODULE=ON -DCMAKE_INSTALL_PREFIX=/usr ..
    make && sudo make install

### Write configs

    cat >> ~/.zshrc << EOF
    if [ -n "$GTK_MODULES" ]; then
        GTK_MODULES="${GTK_MODULES}:unity-gtk-module"
    else
        GTK_MODULES="unity-gtk-module"
    fi

    if [ -z "$UBUNTU_MENUPROXY" ]; then
        UBUNTU_MENUPROXY=1
    fi

    export GTK_MODULES
    export UBUNTU_MENUPROXY
    EOF

    cat >> .config/gtk-3.0/settings.ini < EOF
    gtk-shell-shows-app-menu=true
    gtk-shell-shows-menubar=true
    EOF
