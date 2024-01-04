## Installation and configuration

### Install

#### cryptmypi

- log into your RPi and connect the 32GB SD card via an SD card reader
- you might try to run `cryptmypi.sh` under your favourite Linux OS but I prefer to do it with an RPi and Raspberry Pi OS (Buster) to avoid any cross platform, cross compilation problems

  ```
  $ git clone https://github.com/unixabg/cryptmypi
  # head: https://github.com/unixabg/cryptmypi/commit/52227dfaf905ace65fa8555fb3999e9a75b299f5
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

- upgrade
  ```
  $ sudo apt update && sudo apt upgrade -y
  # check kernel version
  ls /lib/modules
  update-initramfs -u
  # replace "5.10.103-v7l+" with the proper version, refer to the output of the previous ls command
  $ sudo mkinitramfs -o /boot/initramfs.gz 5.10.103-v7l+
  ```
- `raspi-config` --> set timezone
  - alternative: https://raspberrypi.stackexchange.com/questions/87164/setting-timezone-non-interactively
- use pi user instead of root
  ```
  cp -r /root/.ssh /home/pi/
  chown pi:pi -R /home/pi/.ssh
  chmod 600 /home/pi/.ssh/authorized_keys
  ```
- `sudo apt install zsh -y && chsh -s $(which zsh)`
- overwrite `.zsh_history` using the backup copy
- disable ssh pw auth (disabled by default in cryptmypi)
- deploy ufw: https://wiki.ubuntu.com/UncomplicatedFirewall
  ```
  sudo apt install ufw
  sudo ufw allow 22
  sudo ufw enable
  sudo ufw logging on
  sudo ufw show added
  sudo ufw status
  ```
- copy NAS ssh key to `/root/.ssh` and `/home/pi/.ssh`
  - (running `./backup.sh` will need root priviledges to overwrite directory timestamps)
- TODO: disable bluetooth
  - consider doing this in `stage1-otherscript.sh`
  - https://forums.raspberrypi.com/viewtopic.php?t=290611
- set up EHD
  - format
  - encrypt it with LUKS
  - use `fstab` and `crypttab` to mount and unlock it automatically
  - `sudo cryptsetup luksHeaderBackup /dev/mmcblk0p2 --header-backup-file mmcblk0p2-vulcan-luks-header-backup`
  - https://blog.tinned-software.net/automount-a-luks-encrypted-volume-on-system-start/
  - https://forums.unraid.net/topic/83285-quick-command-to-back-up-all-luks-headers/
  - https://loganmarchione.com/2015/05/encrypted-external-drive-with-luks/
  - https://linuxconfig.org/how-to-use-a-file-as-a-luks-device-key
  - https://www.alibabacloud.com/help/doc-detail/34377.htm
    - necessary for drives bigger than 2TB
  - https://blog.tinned-software.net/automount-a-luks-encrypted-volume-on-system-start/
  ```
  $ cat /etc/fstab
  ...
  /dev/mapper/data-crypt        /mnt/data            ext4    defaults   0       2
  ...
  ```

  ```
  $ cat /etc/crypttab
  ...
  data-crypt        UUID=<UUID> <path to LUKS key> luks
  ```
- msmtp
  ```
  $ sudo apt install msmtp
  $ nano .msmtprc
  $ sudo chown pi:pi .msmtprc
  $ sudo chmod 600 .msmtprc
  $ sudo cp ~/.msmtprc /root/
  $ cat .msmtprc
  # Default values ​​for all accounts.
  defaults
  tls on
  auth on
  tls_starttls on
  tls_trust_file /etc/ssl/certs/ca-certificates.crt
  logfile /var/log/msmtp.log
  
  # Example for a Gmail account
  account gmail
  auth plain
  host smtp.gmail.com
  port 587
  from <gmail acc>
  user <gmail acc>
  password <pw>
  # Set the default account
  account default: gmail
  ```
- copy `backup.sh` to `/home/pi`
- copy `backup-history.sh` to `/home/pi`
- copy `check-for-upgrades.sh` to `/home/pi`
  ```
  $ sudo chmod +x backup.sh
  $ sudo chmod +x backup-history.sh
  $ sudo chmod +x check-for-upgrades.sh
  $ dos2unix backup.sh
  $ dos2unix backup-history.sh
  $ dos2unix check-for-upgrades.sh
  ```
- copy `msg*.txt` to `/home/pi`
- crontab
  ```
  $ sudo crontab -e
  ...
  0 0 * * * /usr/bin/bash -c "/home/pi/check-for-upgrades.sh"

  0 0 * * * /usr/bin/bash -c "/home/pi/backup-history.sh"

  0 2 * * * /usr/bin/bash -c "/home/pi/backup.sh"
  ```
- TODO: install syslog
  - https://rubysash.com/operating-system/linux/setup-rsyslog-client-forwarder-on-raspberry-pi/
  - https://forums.raspberrypi.com/viewtopic.php?t=53841
- TODO: disable unnecessary services
- TODO: check https://medium.com/@s0hax/send-an-email-notification-when-a-user-logs-into-your-raspberry-pi-via-ssh-487bfbeb8877
- TODO: hardening
  - https://www.youtube.com/watch?v=6Ip4stkoLws
  - https://www.youtube.com/watch?v=ukHcTCdOKrc
  - https://www.youtube.com/watch?v=6R8uKdstnts
  - https://www.raspberrypi.org/documentation/configuration/security.md
- TODO: check `systmed-resolved.service`
- TODO: deploy docker containers
  - rpi-monitor
  - healthchecks
  - smokeping
  - snipe-it
  - heimdall
  - diskover
  - endlessh
  - scrutiny
  - watchtower
  - grafana, prometheus
  - pihole
    - https://www.youtube.com/watch?v=FnFtWsZ8IP0
- TODO: check log2ram
- TODO: deploy network monitor
  - https://www.technicallywizardry.com/raspberry-pi-network-monitor/
- TODO: deploy IDS
  - https://www.reddit.com/r/raspberry_pi/comments/np1a8f/building_my_home_intrusion_detection_system/
