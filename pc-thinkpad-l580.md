## Installation and configuration

### Install

- create bootable USB with **Fedora Media Writer**
  - https://fedoraproject.org/workstation/download
- boot from USB (F12)
- installation
  - language: english (US)
  - keyboard: HU
  - time (HU)
  - installation media: Samsung NVMe
  - enable LUKS encryption
  - reclaim space (delete all partitions)

#### Extra M.2 NVMe SSD

##### HW

- KBG40ZMT128G
- https://qubitsandbytes.co.uk/m-2-drives-are-not-all-equal/
- https://forums.lenovo.com/t5/ThinkPad-L-R-and-SL-series-Laptops/L580-extra-SSD-slot/m-p/5222063
- https://www.reddit.com/r/thinkpad/comments/dydih6/t580_wont_turn_on_with_m2_nvme_ssd_in_wwan_slot/

##### SW

- format and create 2 partitions: config (97GB), data (31GB)
  - `sudo fdisk -l`
  - `sudo fdisk /dev/<device>`
    - `m`
    - `n`
    - `p`
    - `w`
- encrypt it with LUKS
- use `fstab` and `crypttab` to mount and unlock it automatically
- https://www.digitalocean.com/community/tutorials/create-a-partition-in-linux
- https://www.cyberciti.biz/security/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/
- https://www.redhat.com/sysadmin/disk-encryption-luks
- https://www.cyberciti.biz/hardware/cryptsetup-add-enable-luks-disk-encryption-keyfile-linux/
- https://www.howtoforge.com/automatically-unlock-luks-encrypted-drives-with-a-keyfile
- https://unix.stackexchange.com/questions/658/linux-how-can-i-view-all-uuids-for-all-available-disks-on-my-system

```
$ cat /etc/fstab
...
/dev/mapper/backup-config /media/backup-config ext4 defaults 0 2
/dev/mapper/backup-data /media/backup-data ext4 defaults 0 2
```

```
$ cat /etc/crypttab
...
backup-config /dev/disk/by-uuid/<UUID> <path to LUKS key> luks
backup-data /dev/disk/by-uuid/<UUID> <path to LUKS key> luks
```

### Setup

#### General

- enable third party repos
- set user profile pic
- set wallpaper
- `sudo dnf upgrade --refresh`
- check updates in **Software** as well (system updates for example)
- copy ssh keys from backup to `~/.ssh/`
- enable ssh-rsa keys
  - ```
    $ cat /home/gemesa/.ssh/config
    Host *
        PubkeyAcceptedKeyTypes=+ssh-rsa
    ```
  - https://www.reddit.com/r/Fedora/comments/jh9iyi/f33_openssh_no_mutual_signature_algorithm/
  - https://confluence.atlassian.com/bitbucketserverkb/ssh-rsa-key-rejected-with-message-no-mutual-signature-algorithm-1026057701.html
  - https://unix.stackexchange.com/questions/630446/ssh-in-fedora-33-error-sign-and-send-pubkey-no-mutual-signature-supported
- configure Dash
  - pin **Terminal**
  - remove **Software**
  - remove **Calendar**
  - Dash layout: **Firefox**, **Terminal**, **Files**, **Settings** + the apps below
- configure **Text Editor**
  - **Text Editor** --> **View Options** --> spaces per tab: 4
  - **Text Editor** --> **Main Menu** --> **Preferences** --> appearance: oblivion
- dark theme + nightlight
  - **Settings** --> **Appearance** --> **Style**
  - **Settings** --> **Night Light**
    - manual schedule
    - color temp: 3/4
- update DNF config (`/etc/dnf/dnf.conf`)
  - `max_parallel_downloads=10`
    - https://www.reddit.com/r/Fedora/comments/bc9pyz/why_is_dnf_so_excruciatingly_slow/
  - `defaultyes=True`
    - https://www.reddit.com/r/Fedora/comments/rpttto/make_y_the_default_action_in_dnf/
- install **Gnome Tweaks**
  - **Tweaks** --> **Windows** --> **Titlebar Buttons**
    - check "Maximize"
    - check "Minimize"
    - set "Placement" to right
- install `zsh`
  - `sudo dnf install zsh -y && chsh -s $(which zsh)`
- overwrite `.zshrc` using the backup copy
  - this was originally copied from the following link and tailored according to personal preferences
    - https://gitlab.com/kalilinux/packages/kali-defaults/-/blob/kali/master/etc/skel/.zshrc
    - https://unix.stackexchange.com/questions/655096/what-zsh-theme-does-kali-use
- overwrite `.zsh_history` using the backup copy
- make journal logs persistent
  - https://www.golinuxcloud.com/enable-persistent-logging-in-systemd-journald/
  - `sudo nano /etc/systemd/journald.conf`
    - `#Storage=auto` --> `Storage=persistent`
  - see logs at `/var/log/journal`
- `sudo dnf install fira-code-fonts`
  - `fc-list : family style | grep "Fira"`
- configure `gnome-terminal`
  - install nord theme
    - https://github.com/arcticicestudio/nord-gnome-terminal
  - set font to fira code (size: 10) at **Terminal** --> **Preferences** --> **Profiles** --> **Nord** --> **Text**
- `sudo dnf install neovim`
  - copy `init.vim` from backup to `~/.config/nvim/`

#### Base apps

- **Virtual Machine Manager**
- **Beyond Compare**
- **VS Code**
- **Docker**
- **NormCap**
- **Wireshark**
- **Ghidra**
- **Remmina**
- **Raspberry Pi Imager**
- **GitKraken**
- **VirtualBox**
- **Burp Suite**
- **Postman**
- **Bless Hex Editor**
- **LibreOffice**

##### Firefox

- enable dark theme
- set Google to english
- disable Firefox "PW manager" at **Firefox** --> **Settings** --> **Privacy & Security** --> **Logins and Passwords**
- sign in (Mozilla account)
  - syncs bookmarks, history etc.
- enable dark pdfs
  - https://stackoverflow.com/questions/61814564/how-can-i-enable-dark-mode-when-viewing-a-pdf-file-in-firefox (see answer from Alfian Hamdani)
- extensions:
  - **Dark Reader**
  - **ESET PW Manager**
  - **Wappalyzer**
  - **Windscribe VPN**

##### WireGuard

`sudo dnf install wireguard-tools`

###### Client config

- use the backup `l580.conf` or generate a new key
- https://openwrt.org/docs/guide-user/services/vpn/wireguard/server
- necessary keys (client config):
  - `wgclient.key`
  - `wgclient.psk`
  - `wgserver.pub`
    - generate the server public key from the server private key
    - **LuCI** --> **Network** --> **Interfaces** --> **VPN** --> **Edit** --> **General Settings** --> **Private Key**
- necessary keys (server config):
  - `wgserver.key`
  - `wgclient.pub`
  - `wgclient.psk`

```
[Interface]
Address = <ipv4 address>, <ipv6 address>
PrivateKey = <content of wgclient.key>
DNS = <DNS address>
[Peer]
PublicKey = <content of wgserver.pub>
PresharedKey = <content of wgclient.psk>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = <(DDNS) domain or IP address>:51820
```

###### Check connection

```
sudo wg-quick up l580
sudo wg show
smbclient //chronos.lan/<shared folder>
sudo wg-quick down l580
```

- https://unix.stackexchange.com/questions/206415/sending-files-over-samba-with-command-line
- https://help.ubuntu.com/community/Samba/SambaClientGuide
- https://unix.stackexchange.com/questions/285941/smbclient-asking-for-password

##### Timeshift

`sudo dnf install timeshift`

- **Timeshift** --> **Settings** --> **Type**
  - rsync
- **Timeshift** --> **Settings** --> **Location**
  - nvme1n1p1/dm-1 (97GB)
- **Timeshift** --> **Settings** --> **Schedule**
  - boot, keep 3
- **Timeshift** --> **Settings** --> **Filters**
  - `/home/**`
  - `/var/lib/libvirt/**`
  - `/root/**`

##### Borg

`sudo dnf install borgbackup`

use the script from https://borgbackup.readthedocs.io/en/stable/quickstart.html#automating-backups with some modifications:

```
...
export BORG_REPO=/media/backup-data/borg
...
#export BORG_PASSPHRASE='XYZl0ngandsecurepa_55_phrasea&&123'
...
    --exclude 'home/*/.cache/*'             \
    --exclude 'home/*/.mozilla/*'           \
    --exclude 'root/.cache/*'               \
    --exclude 'home/*/.local/share/Trash/*' \
    --exclude '*.iso'                       \
    --exlcude '*.zip'                       \
    --exclude 'home/*/git-repos/*'          \
                                            \
    ::'{hostname}-{now}'                    \
    /home                                   \
    /root                                   \
    /var/log
...
```
```
sudo crontab -e
/30 * * * * <path to borg.sh> >> <path to borg.log> 2>&1
```

#### Optional apps

- **CLion**
- **PyCharm**
- **Geany**
- **STM32CubeIDE**
- **VLC**
- **GIMP**
- **Pinta**

#### Other packages

```
sudo dnf install valgrind
sudo dnf install bmon
sudo dnf install nmap
sudo dnf install p0f
sudo dnf install hashcat
sudo dnf install aircrack-ng
sudo dnf install shellcheck
sudo dnf install shfmt
sudo dnf install tftp-server tftp
sudo dnf install gpsbabel
sudo dnf install make autoconf automake cmake ninja-build ccache gcc gcc-c++ kernel-devel
sudo dnf install clang
sudo dnf install git curl wget python3
sudo dnf install coreutils
sudo dnf install gdb
sudo dnf install minicom
sudo dnf install xfreerdp
sudo dnf install hydra
sudo dnf install htop
sudo dnf install upx
sudo dnf install hexedit
sudo dnf install nasm
sudo dnf install gobuster
sudo dnf install mingw64-gcc
sudo dnf install strace
sudo dnf install musl-tools
sudo dnf install pandoc
```

#### Manual installation

- https://www.rust-lang.org/tools/install
- https://github.com/ZerBea/hcxdumptool
- https://github.com/ZerBea/hcxtools
- https://github.com/openwall/john
- https://github.com/klauspost/asmfmt
- https://github.com/frida/frida
- https://github.com/rapid7/metasploit-framework
- https://github.com/blackberry/pe_tree
- https://github.com/volatilityfoundation/volatility3
