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
  - remove **Calendar**
  - Dash layout: **Firefox**, **Terminal**, **Files**, **Settings**, **Software** + the apps below
- configure **Text Editor**
  - **Text Editor** --> **Main Menu** --> **Preferences** --> indentation: character: space, spaces per tab: 4
  - **Text Editor** --> **Main Menu** --> **Preferences** --> appearance: oblivion
- dark theme + nightlight
  - **Settings** --> **Appearance** --> **Style**
  - **Settings** --> **Displays**
    - --> **Night Light**
      - manual schedule: 2 am - 2 am
      - color temp: 3/4
    - --> Scale: 100%
- update DNF config (`/etc/dnf/dnf.conf`)
  - `max_parallel_downloads=10`
    - https://www.reddit.com/r/Fedora/comments/bc9pyz/why_is_dnf_so_excruciatingly_slow/
  - `defaultyes=True`
    - https://www.reddit.com/r/Fedora/comments/rpttto/make_y_the_default_action_in_dnf/
- install **Gnome Tweaks** from **Software**
  - install Adwaita-dark theme
    - `sudo dnf install adw-gtk3-theme`
  - **Tweaks** --> **Appearance** --> **Styles** --> **Legacy Applications**
    - set `Adw-gtk3-dark`
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
- terminal (`ptyxis-agent`)
  - **Preferences** --> **Appearance** --> Nord

#### ssh

- copy ssh keys from backup to `~/.ssh/`
- enable ssh-rsa keys

```
$ nano /home/gemesa/.ssh/config
$ cat /home/gemesa/.ssh/config
Host *
    PubkeyAcceptedKeyTypes=+ssh-rsa
```

Allow SHA1 temporarily if necessary:

```
$ update-crypto-policies --show
DEFAULT
$ sudo update-crypto-policies --set DEFAULT:SHA1
# run the cmd ... then revert the policy change
$ sudo update-crypto-policies --set DEFAULT
# reboot
$ update-crypto-policies --show
DEFAULT
```

- https://www.reddit.com/r/Fedora/comments/jh9iyi/f33_openssh_no_mutual_signature_algorithm/
- https://confluence.atlassian.com/bitbucketserverkb/ssh-rsa-key-rejected-with-message-no-mutual-signature-algorithm-1026057701.html
- https://unix.stackexchange.com/questions/630446/ssh-in-fedora-33-error-sign-and-send-pubkey-no-mutual-signature-supported

#### gpg

- copy gpg keys from backup to `~/`

```
$ gpg --import ~/private.asc
$ gpg --list-secret-keys --keyid-format=long
$ rm ~/private.asc
```

#### Apps

##### Beyond Compare

- https://www.scootersoftware.com/download
- https://www.scootersoftware.com/kb/linux_install#redhat

```
$ wget https://www.scootersoftware.com/files/bcompare-5.2.6.32774.x86_64.rpm
$ sudo dnf install bcompare-5.2.6.32774.x86_64.rpm
```

##### Docker

- https://docs.docker.com/engine/install/fedora/

```
$ sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
$ sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
$ sudo systemctl enable --now docker
$ sudo docker run hello-world
```

##### Ente Auth

- https://github.com/ente/ente#ente-auth
- https://github.com/ente/ente/releases?q=tag%3Aauth-v4

```
$ sudo dnf install libayatana-appindicator-gtk3
$ wget https://github.com/ente/ente/releases/download/auth-v4.4.25/ente-auth-v4.4.25-x86_64.rpm
$ sudo dnf install ./ente-auth-v4.4.25-x86_64.rpm
```

##### Etcher

- https://github.com/balena-io/etcher#redhat-rhel-and-fedora-based-package-repository-gnulinux-x86x64

```
$ wget https://github.com/balena-io/etcher/releases/download/v2.1.7/balena-etcher-2.1.7-1.x86_64.rpm
$ sudo dnf install balena-etcher-2.1.7-1.x86_64.rpm
```

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

##### Foliate

- install via **Software**

##### git

```
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

##### LibreOffice

- install via **Software**

##### Malware analysis tools

- https://github.com/gemesa/malware-analysis-toolkit

##### Metasploit

- https://docs.metasploit.com/docs/using-metasploit/getting-started/nightly-installers.html#installing-metasploit-on-linux--macos

##### nano

```
$ nano ~/.nanorc
$ cat ~/.nanorc
set constantshow
```

##### NordVPN

- https://nordvpn.com/download/linux/

```
$ sh <(wget -qO - https://downloads.nordcdn.com/apps/linux/install.sh) -p nordvpn-gui
```

##### NormCap

- install via **Software**

##### Raspberry Pi Imager

- install via **Software**

##### Rust

- https://www.rust-lang.org/tools/install

##### Tailscale

- [setup](router.md#tailscale)

##### tealdeer

- https://github.com/tealdeer-rs/tealdeer

```
$ cargo install tealdeer
```

##### VS Code

- https://code.visualstudio.com/docs/setup/linux
- **File** --> **Preferences** --> **Settings**
  - --> **Text Editor**
    - --> **Files**
      - Auto Save: afterDelay
    - --> **Tab Size**: 4
  - --> **Chat** --> **Miscellaneous** --> Disable AI Features
- **Word Wrap** --> on
- **Extensions**
  - **Dependi**
  - **Even Better TOML**
  - **Hex Editor**
  - **LLVM** (LLVM IR syntax highlight)
  - **Markdown PDF**
  - **Nord**
  - **rust-analyzer**
  - **YARA Language**

##### yazi

References:

- https://yazi-rs.github.io/docs/installation#cargo-binstall

```
$ cargo install cargo-binstall
$ cargo binstall yazi-fm yazi-cli
# replace v3.5.1 with the latest release
$ wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.5.1/NerdFontsSymbolsOnly.tar.xz
$ tar -xf NerdFontsSymbolsOnly.tar.xz
$ mkdir -p ~/.local/share/fonts
$ mv *.ttf ~/.local/share/fonts/
$ fc-cache -fv
$ fc-list | grep -i nerd
# restart terminal
```

#### Optional apps

##### GIMP

- install via **Software**

##### Pinta

- install via **Software**

##### VLC

- install via **Software**
