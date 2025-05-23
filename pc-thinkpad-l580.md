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

- format and create a partition
  - `sudo fdisk -l`
  - `sudo fdisk /dev/sda` --> replace `/dev/sda` with the correct device
    - `m`
    - `n`
    - `p`
    - `w`
  - `sudo blkid /dev/sda1`
    - make a note of the uuid
- encrypt, create filesystem and mountpoint
  - `sudo cryptsetup luksFormat --type luks2 /dev/sda1`
  - `dd bs=512 count=4 if=/dev/random of=luks-hdd.key`
  - `sudo cryptsetup luksOpen /dev/sda1 hdd`
  - `sudo cryptsetup luksAddKey /dev/sda1 luks-hdd.key`
  - `sudo cryptsetup luksClose hdd`
  - `sudo cryptsetup luksOpen /dev/sda1 hdd --key-file=luks-hdd.key`
  - `sudo mkfs.ext4 /dev/mapper/hdd`
  - `sudo mkdir /mnt/hdd`
  - `sudo mount /dev/mapper/hdd /mnt/hdd`
  - `df -H /mnt/hdd`
  - `sudo umount /mnt/hdd`
  - `sudo cryptsetup luksClose hdd`
- use `fstab` and `crypttab` to mount and unlock it automatically

```
$ cat /etc/fstab
...
/dev/mapper/nvme2 /mnt/nvme2 ext4 defaults,nofail 0 2
```

```
$ cat /etc/crypttab
...
nvme2 /dev/disk/by-uuid/<UUID> <path to LUKS key> luks,nofail
```
###### References

- https://www.digitalocean.com/community/tutorials/create-a-partition-in-linux
- https://www.cyberciti.biz/security/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/
- https://www.redhat.com/sysadmin/disk-encryption-luks
- https://www.cyberciti.biz/hardware/cryptsetup-add-enable-luks-disk-encryption-keyfile-linux/
- https://www.howtoforge.com/automatically-unlock-luks-encrypted-drives-with-a-keyfile
- https://unix.stackexchange.com/questions/658/linux-how-can-i-view-all-uuids-for-all-available-disks-on-my-system
- https://forums.linuxmint.com/viewtopic.php?t=352620

### Setup

#### General

- enable third party repos
- set user profile pic
- set wallpaper
- `sudo dnf upgrade --refresh`
- check updates in **Software** as well (system updates for example)
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
- install **Gnome Tweaks** from **Software**
  - install Adwaita-dark theme
    - `sudo dnf install gnome-themes-extra`
  - **Tweaks** --> **Appearance** --> **Styles** --> **Legacy Applications**
    - set Adwaita-dark
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

#### ssh

- copy ssh keys from backup to `~/.ssh/`
- enable ssh-rsa keys

```
$ nano /home/gemesa/.ssh/config
$ cat /home/gemesa/.ssh/config
Host *
    PubkeyAcceptedKeyTypes=+ssh-rsa
```

- https://www.reddit.com/r/Fedora/comments/jh9iyi/f33_openssh_no_mutual_signature_algorithm/
- https://confluence.atlassian.com/bitbucketserverkb/ssh-rsa-key-rejected-with-message-no-mutual-signature-algorithm-1026057701.html
- https://unix.stackexchange.com/questions/630446/ssh-in-fedora-33-error-sign-and-send-pubkey-no-mutual-signature-supported

#### Base apps

Plain install:
- **[Beyond Compare](https://www.scootersoftware.com/download)**
- **[Docker](https://docs.docker.com/engine/install/fedora/)**
- **[Ente Auth](https://github.com/ente-io/ente/releases)**
- **[Etcher](https://etcher.balena.io/#download-etcher)**
- **Foliate** via **Software**
- **LibreOffice** via **Software**
- **[NordVPN](https://nordvpn.com/download/linux/#install-nordvpn)**
- **NormCap** via **Software**
- **Raspberry Pi Imager** via **Software**
- **[tealdeer](https://github.com/tealdeer-rs/tealdeer)**

##### Firefox

- sign in (Mozilla account)
  - syncs bookmarks, history etc. and the following settings
- pin extensions to toolbar

###### Settings (should be synced automatically)

- enable dark theme
- set Google to english
- disable Firefox "PW manager" at **Firefox** --> **Settings** --> **Privacy & Security** --> **Logins and Passwords**
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

##### VS Code

- https://code.visualstudio.com/docs/setup/linux
- **File** --> **Preferences** --> **Settings** --> **Text Editor**
  - --> **Font**
    - Font size: 12
    - Font family: Fira Code
  - --> **Files**
    - Auto Save: afterDelay
- **Extensions**
  - **Assembly for ARM64**
  - **CodeLLDB**
  - **Dependi**
  - **Even Better TOML**
  - **Markdown PDF**
  - **Nord**
  - **rust-analyzer**
  - **Scala (Metals)**
  - **YARA**

##### Malware analysis tools

- https://github.com/gemesa/malware-analysis-toolkit

##### Rust

- https://www.rust-lang.org/tools/install

##### Metasploit

- https://docs.metasploit.com/docs/using-metasploit/getting-started/nightly-installers.html#installing-metasploit-on-linux--macos

##### git

```
$ gpg --import private.gpg
$ gpg --list-secret-keys --keyid-format=long
$ nano ~/.ssh/config
$ cat ~/.ssh/config
IdentityFile ~/.ssh/id_ed25519_github
$ sudo chmod 600 ~/.ssh/config
$ git config --global user.name "Andras Gemes"
$ git config --global user.email "gemesa@protonmail.com"
$ git config --global commit.gpgsign true
$ git config --global user.signingkey <key ID>
$ git config --global core.editor "nano"
$ git config --global --list
```

##### nano

```
$ nano ~/.nanorc
$ cat ~/.nanorc
set constantshow
```

##### yazi

```
$ cargo install --locked --git https://github.com/sxyazi/yazi.git yazi-fm yazi-cli
# replace v3.2.1 with the latest release
$ wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/NerdFontsSymbolsOnly.tar.xz
$ tar -xf NerdFontsSymbolsOnly.tar.xz
$ mkdir -p ~/.local/share/fonts
$ mv *.ttf ~/.local/share/fonts/
$ fc-cache -fv
$ fc-list | grep -i nerd
# restart terminal
```

how to change font in WSL:
- download on the host: https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/FiraCode.zip
- double click on FiraCodeNerdFont-Regular.ttf and install it
- right click in WSL --> Settings --> Profiles --> Default
  - --> color scheme: one half dark
  - --> font face: firacode nerd font
  - --> font size: 10

##### Waydroid

```
$ sudo waydroid init -s GAPPS -f -chttps://ota.waydro.id/system -v https://ota.waydro.id/vendor
$ sudo firewall-cmd --add-port=53/udp --permanent
$ sudo firewall-cmd --add-port=67/udp --permanent
$ sudo firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -j ACCEPT
$ sudo firewall-cmd --permanent --direct --add-rule ipv6 filter FORWARD 0 -j ACCEPT
$ firewall-cmd --reload
$ systemctl status waydroid-container.service
$ waydroid session start
$ waydroid show-full-ui
```

- https://docs.waydro.id/usage/install-on-desktops#fedora
- https://docs.waydro.id/faq/google-play-certification
- https://www.reddit.com/r/waydroid/comments/z57rx7/help_i_cant_connect_waydroid_to_internet/

#### Optional apps

Plain install:
- **GIMP** via **Software**
- **Pinta** via **Software**
- **VLC** via **Software**
