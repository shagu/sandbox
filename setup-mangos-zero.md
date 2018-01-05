Setup MaNGOS-Zero
=================

### Prepare LXC

    lxc-create -n mangos-zero -t ubuntu

    # open /var/lib/lxc/mangos-zero/config
    # change 'lxc.network.type' to 'none' to use host network

    # start the container
    lxc-start -n mangos-zero

    # attach the container
    lxc-attach -n mangos-zero

    # add DNS server
    echo "nameserver 8.8.8.8" > /etc/resolv.conf

    # install database and webfrontend
    apt-get update

    apt-get install phpmyadmin mariadb-server
    mysql_secure_installation

### install build dependencies

    apt-get install git curl autoconf automake cmake libtool build-essential \
      libbz2-dev libace-dev libssl-dev libmysqlclient-dev

### add user and prepare sources

    useradd -m mangos
    passwd mangos
    su -l mangos
    git clone https://github.com/mangoszero/server.git --recursive
    git clone https://github.com/mangoszero/database.git --recursive


### optional: ike playerbots

    cd server
    git fetch https://github.com/ike3/mangosbot-zero mangos-zero-ai:ike3
    git merge ike3
    cd ..

### build the server

    mkdir server/build && cd server/build

    # cmake defaults
    # -DDEBUG=0
    # -DSCRIPT_LIB_ELUNA=1
    # -DSCRIPT_LIB_SD3=1
    # -DPOSTGRESQL=0
    # -DPLAYERBOTS=0
    # -DSOAP=0

    cmake .. \
        -DBUILD_TOOLS=1 \
        -DCMAKE_INSTALL_PREFIX=/home/mangos/run \
        -DPLAYERBOTS=1 \
        -DSCRIPT_LIB_ELUNA=0

    make -j4
    make install
    cd

### install databases

    sudo mysql -u root < database/World/Setup/mangosdCreateDB.sql

    sudo mysql -u root realmd < database/Realm/Setup/realmdLoadDB.sql
    sudo mysql -u root character0 < database/Character/Setup/characterLoadDB.sql
    sudo mysql -u root mangos0 < database/World/Setup/mangosdLoadDB.sql

    sudo mysql -u root realmd < database/Tools/updateRealm.sql

    for i in database/World/Setup/FullDB/*.sql; do
      sudo mysql -u root mangos0 < $i
    done

    # perform updates to match the master branch
    for i in database/World/Updates/Rel21/*/*.sql; do
      sudo mysql -u root mangos0 < $i
    done

### extract gameclient files

    # copy client to ~/gameclient
    cd gameclient
    ../run/bin/tools/map-extractor
    ../run/bin/tools/vmap-extractor
    ../run/bin/tools/movemap-generator
    cp -rf dbc maps mmaps vmaps ../run/bin

### configure the server

    cd ../run/etc
    cp realmd.conf.dist realmd.conf
    cp mangosd.conf.dist mangosd.conf

    sed -i "s/127.0.0.1;3306;root;mangos;/127.0.0.1;3306;mangos;mangos;/g" mangosd.conf

### create systemd-services

    cat > /etc/systemd/system/realmd.service << _EOF_
    [Unit]
    Description=MaNGOS Realm Server
    After=mysql

    [Install]
    WantedBy=multi-user.target

    [Service]
    User=mangos
    Group=mangos

    WorkingDirectory=/home/mangos/run/bin
    ExecStart=/home/mangos/run/bin/realmd

    Restart=always
    RestartSec=5s
    _EOF_

    cat > /etc/systemd/system/mangosd.service << _EOF_
    [Unit]
    Description=MaNGOS Game Server
    After=mysql

    [Install]
    WantedBy=multi-user.target

    [Service]
    User=mangos
    Group=mangos

    WorkingDirectory=/home/mangos/run/bin
    ExecStart=/home/mangos/run/bin/mangosd

    Restart=always
    RestartSec=5s
    _EOF_

    sed -i "s/Console.Enable = 1/Console.Enable = 0/g" /home/mangos/run/etc/mangosd.conf
    systemctl enable realmd
    systemctl enable mangosd
