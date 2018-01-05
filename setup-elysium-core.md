Setup Elysium/LightsHope Core
=============================

### Prepare LXC

    lxc-create -n elysium -t ubuntu

    # open /var/lib/lxc/elysium/config
    # change 'lxc.network.type' to 'none' to use host network

    lxc-start -n elysium
    lxc-attach -n elysium

    echo "nameserver 8.8.8.8" >> /etc/resolv.conf

    apt-get update
    apt-get install vim

    apt-get install phpmyadmin mariadb-server
    mysql_secure_installation

### install build dependencies

    apt-get install libace-dev git  build-essential gcc g++ automake autoconf make \
                    patch libmysql++-dev libtool libssl-dev grep binutils zlibc \
                    libc6 libbz2-dev cmake libboost-dev libboost-system-dev \
                    libboost-program-options-dev libboost-thread-dev libboost-regex-dev \
                    libtbb-dev

### add user and prepare sources

    useradd -m elysium
    passwd elysium
    su -l elysium

    git clone https://github.com/elysium-project/server.git server
    git clone https://github.com/elysium-project/database.git database

### build the server

    cd server
    git checkout master
    mkdir build && cd build

    cmake .. \
        -DUSE_ANTICHEAT=0 \
        -DCMAKE_INSTALL_PREFIX=/home/cmangos/run

    # [TOOD] broken extractors: -DUSE_EXTRACTORS=1 \

    make -j4
    make install

### install databases

    sudo mysql -u root -p < mangos/sql/create/db_create_mysql.sql
    sudo mysql -u root -p characters < mangos/sql/base/characters.sql
    sudo mysql -u root -p mangos < mangos/sql/base/mangos.sql
    sudo mysql -u root -p realmd < mangos/sql/base/realmd.sql
    sudo mysql -u root -p mangos < mangos/sql/scriptdev2/scriptdev2.sql
    sudo mysql -u root -p mangos < classicdb/Full_DB/ClassicDB_*.sql

### extract gameclient files

    # copy client to ~/gameclient
    cd gameclient
    mkdir vmaps mmaps
    ../run/bin/tools/ad
    ../run/bin/tools/vmap_extractor
    ../run/bin/tools/vmap_assembler Buildings vmaps
    ../run/bin/tools/MoveMapGen
    cp -rf dbc maps mmaps vmaps ../run/bin

### configure the server

    cd ../run/etc
    cp realmd.conf.dist realmd.conf
    cp mangosd.conf.dist mangosd.conf

### create systemd-services

    cat > /etc/systemd/system/realmd.service << _EOF_
    [Unit]
    Description=CMaNGOS Realm Server
    After=mysql

    [Install]
    WantedBy=multi-user.target

    [Service]
    User=cmangos
    Group=cmangos

    WorkingDirectory=/home/cmangos/run/bin
    ExecStart=/home/cmangos/run/bin/realmd

    Restart=always
    RestartSec=5s
    _EOF_

    cat > /etc/systemd/system/mangosd.service << _EOF_
    [Unit]
    Description=CMaNGOS Game Server
    After=mysql

    [Install]
    WantedBy=multi-user.target

    [Service]
    User=cmangos
    Group=cmangos

    WorkingDirectory=/home/cmangos/run/bin
    ExecStart=/home/cmangos/run/bin/mangosd

    Restart=always
    RestartSec=5s
    _EOF_

    sed -i "s/Console.Enable = 1/Console.Enable = 0/g" /home/cmangos/run/etc/mangosd.conf
    systemctl enable realmd
    systemctl enable mangosd

