## Installation and configuration

### Install

#### cryptmypi

- log into your RPi and connect the 32GB SD card via an SD card reader
- you might try to run `cryptmypi.sh` under your favourite Linux OS but I prefer to do it with an RPi and Raspberry Pi OS (Buster) to avoid any cross platform, cross compilation problems

  ```
  $ git clone https://github.com/unixabg/cryptmypi
  $ cd cryptmypi
  $ nano examples/pios-encrypted-basic-dropbear/cryptmypi.conf
  # edit the following lines:
  #
  # RPi 4 B 4GB --> v7l+
  # export _KERNEL_VERSION_FILTER="v7l+"
  #
  # export _HOSTNAME="vulcan"
  #
  # check with fdisk -l
  # export _BLKDEV="/dev/sda"
  #
  # export _LUKSPASSWD="<LUKS pw>"
  #
  # use the latest buster image:
  # export _IMAGEURL="https://downloads.raspberrypi.com/raspios_lite_armhf/images/raspios_lite_armhf-2021-05-28/2021-05-07-raspios-buster-armhf-lite.zip"
  # export _IMAGESHA="c5dad159a2775c687e9281b1a0e586f7471690ae28f2f2282c90e7d59f64273c"
  #
  # currently only RSA keys are supported (no Ed25519 for example)
  # export _SSH_LOCAL_KEYFILE="$_USER_HOME/.ssh/id_rsa"
  #
  # export _ROOTPASSWD="<root pw>"
  ```
  note:

  ```
  # my experience is that newer images like bullseye break after doing a kernel upgrade (even if I rebuild initramfs):
  $ sudo apt update && sudo apt upgrade -y
  # check kernel version
  $ ls /lib/modules
  update-initramfs -u
  # tried "update-initramfs -u -k 6.1.21-v7l+" as well, did not help
  # replace "6.1.21-v7l+" with the proper version, refer to the output of the previous ls command
  $ sudo mkinitramfs -o /boot/initramfs.gz 6.1.21-v7l+
  $ sudo reboot
  $ ssh root@192.168.1.102 -i ~/.ssh/id_rsa -p 2222
  Enter passphrase for /dev/disk/by-uuid/<UUID>: 
  Cannot initialize device-mapper. Is dm_mod kernel module loaded?
  Cannot use device crypt, name is invalid or still in use.
  ```
  refs:
    - https://raspberrypi.stackexchange.com/questions/108375/raspbian-dm-mod-missing-after-mkinitramfs-on-luks-encrypted-partition
    - https://unix.stackexchange.com/questions/327914/initramfs-luks-and-dm-mod-cant-boot-after-upgrade

- run the script and go through the interactive installer:
  ```
  $ ./cryptmypi.sh examples/pios-encrypted-basic-dropbear
  ```

- insert the SD card into the RPi and then power it on
- unlock the RPi (this might not work for the first time for some reason, do a power reset in this case and try again)
  ```
  $ ssh root@192.168.1.102 -i ~/.ssh/id_rsa -p 2222
  Enter passphrase for /dev/disk/by-uuid/<UUID>: 
  Connection to 192.168.1.102 closed.
  ```
- log in
  ```
  $ ssh root@192.168.1.102 -i ~/.ssh/id_rsa
  Linux vulcan 5.10.103-v7l+ #1529 SMP Tue Mar 8 12:24:00 GMT 2022 armv7l

  The programs included with the Debian GNU/Linux system are free software;
  the exact distribution terms for each program are described in the
  individual files in /usr/share/doc/*/copyright.

  Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
  permitted by applicable law.
  Last login: Wed Jan  3 19:42:32 2024 from 192.168.2.157

  Wi-Fi is currently blocked by rfkill.
  Use raspi-config to set the country before use.

  root@vulcan:~#
  ```

### Config

TODO: add configuration details
