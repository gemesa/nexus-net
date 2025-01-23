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
- **Foliate** via **Software**
- **LibreOffice** via **Software**
- **[NordVPN](https://nordvpn.com/download/linux/#install-nordvpn)**
- **NormCap** via **Software**
- **Raspberry Pi Imager** via **Software**
- **Remmina** via `dnf`
- **[Wireshark](https://fedoramagazine.org/how-to-install-wireshark-fedora/)**

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
  - **x86 and x86_64 Assembly**

##### Ghidra

- install [from source](https://github.com/NationalSecurityAgency/ghidra/)
  - install [sdkman](https://sdkman.io/)
  - `curl -s "https://get.sdkman.io" | bash`
  - `sdk install gradle`
  - `sdk list java`
  - `sdk install java 21.0.4-tem`
  - `gradle -I gradle/support/fetchDependencies.gradle`
  - `gradle buildGhidra`
- **Edit** --> **Theme** --> **Switch...** --> Flat Dark
- install extensions
  - https://github.com/google/binexport
  - https://github.com/ubfx/BinDiffHelper

##### DIE

- [enable dark theme for Qt applications](https://unix.stackexchange.com/questions/745499/how-to-enable-dark-theme-for-qt-applications)
- install [from source](https://github.com/horsicq/Detect-It-Easy/blob/master/docs/BUILD.md#how-to-build-with-cmake)
- prerequisites:
  - `sudo dnf install cmake qt5-qtbase-devel qt5-qttools-devel qt5-qtmultimedia-devel qt5-qtsvg-devel qt5-qtwebsockets-devel qt5-qtdeclarative-devel qt5-qtscript-devel qt5-qtquickcontrols2-devel qt5-qtwayland-devel`

##### Python

- `sudo dnf install python pip`

##### capa

- [download](https://github.com/mandiant/capa/releases) and add to path

##### OpenSSL

- `sudo dnf install openssl`

##### LLVM

- `sudo dnf install llvm`
- contains `clang` and LLVM binutils (such as `llvm-objdump`)

##### Rust

- https://www.rust-lang.org/tools/install

##### frida

- `pip install frida-tools`

##### Metasploit

- https://docs.metasploit.com/docs/using-metasploit/getting-started/nightly-installers.html#installing-metasploit-on-linux--macos

##### Virtual Machine Manager

- install from **Software**
- https://fedoramagazine.org/full-virtualization-system-on-fedora-workstation-30/
- create Ubuntu 22.04 VM
    - **Virtual Machine Manager** --> Ubuntu VM --> **Open** --> **Show virtual hardware details** --> **Video** --> select VGA or Virtio (disable 3D acceleration)
    - Ubuntu VM --> **Settings** --> **Displays** --> resolution: 1920x1080
        - https://superuser.com/questions/132322/how-to-increase-the-visualized-screen-resolution-on-qemu-kvm
- set up shared folder
    - https://blog.sergeantbiggs.net/posts/file-sharing-with-qemu-and-virt-manager/
    - https://nts.strzibny.name/how-to-set-up-shared-folders-in-virt-manager/

##### VirtualBox

- prerequisites
  - `sudo dnf install gcc make perl kernel-devel`
- https://www.virtualbox.org/wiki/Linux_Downloads
  - refer to chapter "RPM-based Linux distributions" and install from the repo
  - download `oracle_vbox_2016.asc`
  - `sudo rpm --import oracle_vbox_2016.asc`
  - add the [Fedora repo file](https://download.virtualbox.org/virtualbox/rpm/fedora/virtualbox.repo) to `/etc/yum.repos.d/`
  - `sudo dnf install VirtualBox-7.0` (install latest major version)
  - `sudo usermod -a -G vboxusers $USER`
- in case of error `VirtualBox can't enable the AMD-V extension. Please disable the KVM kernel extension, recompile your kernel and reboot (VERR_SVM_IN_USE).`
  ```
  $ lsmod | grep kvm
  kvm_amd               249856  0
  kvm                  1449984  1 kvm_amd
  $ sudo modprobe -r kvm_amd
  ```

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

##### Fedora RE VM (in VirtualBox)

- download and install [Fedora](https://fedoraproject.org/workstation/download)
- machine settings:
  - **Settings** --> **System**
    - --> **Processor** --> Processors: 4
    - --> **Motherboard** --> Base Memory: 4096
  - **Devices** --> **Shared Clipboard** --> Bidirectional
- **Settings**
  - --> **Display** --> **Resolution** --> 1600 x 900
  - --> **Appearance** --> **Style** --> Dark
- `sudo dnf upgrade --refresh -y`
- install [Python](#python)
- install [LLVM](#llvm)
- install [OpenSSL](#openssl)
- install [DIE](#die)
- install [capa](#capa)
- install [Ghidra](#ghidra)
- machine settings:
  - **Settings**
    - --> **Adapter 1** --> Attached to: Internal Network
    - --> **Shared Folders** --> add read-only, auto-mount folder, path: `/home/gemesa/Downloads/malware-bazaar`
- take VM snapshot

#### Optional apps

Plain install:
- **GIMP** via **Software**
- **Pinta** via **Software**
- **VLC** via **Software**

##### JetBrains Toolbox: CLion/PyCharm

- **File** --> **Settings**
  - **Appearance & Behaviour** --> **New UI** --> Enable new UI
  - **Tools** --> **Python Integrated Tools** --> **Docstrings** --> Google format
  - **Plugins** --> **Nord**
  - **Editor** --> **Font**
    - Font: Fira code
    - Font size: 12
    - Line height: 1.2
