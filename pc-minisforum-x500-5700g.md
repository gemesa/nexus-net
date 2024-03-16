## Installation and configuration

### Install

Refer to [pc-thinkpad-l580.md](pc-thinkpad-l580.md#install)

#### Extra Seagate BarraCuda 2.5" 2TB

##### HW

- https://www.seagate.com/gb/en/products/hard-drives/barracuda-2-5-hard-drive/

##### SW

Refer to [pc-thinkpad-l580.md](pc-thinkpad-l580.md#sw)

```
$ cat /etc/fstab
...
/dev/mapper/hdd /mnt/hdd ext4 defaults 0 2
```

```
$ cat /etc/crypttab
...
hdd /dev/disk/by-uuid/<UUID> <path to LUKS key> luks
```

### Setup

Refer to [pc-thinkpad-l580.md](pc-thinkpad-l580.md#setup)
