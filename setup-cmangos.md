Setup CMaNGOS
=============

### Prepare LXC

    lxc-create -n cmangos -t ubuntu

    # open /var/lib/lxc/cmangos/config
    # change 'lxc.network.type' to 'none' to use host network

    lxc-start -n cmangos
    lxc-attach -n cmangos

    echo "nameserver 8.8.8.8" >> /etc/resolv.conf

    apt-get update
    apt-get install vim

    apt-get install phpmyadmin mariadb-server
    mysql_secure_installation

### install build dependencies

    apt-get install libace-dev git  build-essential gcc g++ automake autoconf make \
                    patch libmysql++-dev libtool libssl-dev grep binutils zlibc \
                    libc6 libbz2-dev cmake libboost-dev libboost-system-dev \
                    libboost-program-options-dev libboost-thread-dev libboost-regex-dev

### add user and prepare sources

    useradd -m cmangos
    passwd cmangos
    su -l cmangos
    git clone git://github.com/cmangos/mangos-classic.git mangos
    git clone https://github.com/cmangos/classic-db.git classicdb

### build the server

    mkdir mangos/build && cd mangos/build
    cmake .. \
        -DBUILD_EXTRACTORS=1 \
        -DCMAKE_INSTALL_PREFIX=/home/cmangos/run
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

