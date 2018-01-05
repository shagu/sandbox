Mount RAW-Image Paritions
=========================

* depends: `kpartx`, `dmsetup`

### Commands

    losetup /dev/loop0 raw-image.dd
    echo "0 `blockdev --getsize /dev/loop0` linear /dev/loop0 0" | dmsetup create sdx
    kpartx -a /dev/mapper/sdx
    mount /dev/mapper/sdx1 /mnt
