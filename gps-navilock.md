NAVILOCK GPS NL-454US USB
=========================

### Setup the device

    stty 38400 raw </dev/ttyUSB2
    gpsd -D 5 -N -n /dev/ttyUSB2
    # or
    gpsd /dev/ttyUSB2

### Show satellites

    xgps
