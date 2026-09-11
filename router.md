## Installation and configuration

### Install OpenWrt firmware

#### Netgear Nighthawk X4S R7800

https://openwrt.org/toh/netgear/r7800

#### ASUS RT-AX53U

https://shadowshell.io/flash-openwrt-on-your-asus-rt-ax53u-router

#### TP-Link Archer C7

https://shadowshell.io/unbrick-your-tp-link-archer-c7-openwrt-router

### Upgrade using Attended Sysupgrade

https://openwrt.org/docs/guide-user/installation/attended.sysupgrade

### Generic configuration

- access: http://192.168.1.1
- **System**
  - --> **Administration**
    - --> **Router Password**: set root pw
    - --> **HTTP(S) Access**
      - **Redirect to HTTPS**: enable
    - --> **SSH Access**
      - **Interface**: LAN
      - **Password authentication**: disable
      - **Allow root logins with password**: disable
    - --> **SSH-Keys**
      - generate ssh keys and add public key
      - `ssh-keygen -t ed25519`
  - --> **System**
    - **Hostname**: helios (access: https://helios.lan)
    - **Timezone**
  - --> **Software**
    - run **Update lists...**
    - install packages
      - `nano`
      - `fdisk`
      - `lsblk`
      - `pciutils`
      - `usbutils`
      - `luci-app-statistics`

### Network

- **Network**
  - --> **Firewall** --> **General Settings**
    - **Drop invalid packets**: enable
    - **Software flow offloading**: enable
  - --> **Interfaces**
    - *optional:* edit `lan` IPv4 address (192.168.2.1 / 192.168.3.1 / ...)

### Wireless

- **Network** --> **Wireless**
- ESSID: Helios
- ESSID: Helios-5G
- country code: HU
- encryption: WPA2-PSK/WPA3-SAE Mixed mode
- enable KRACK countermeasures
- 802.11w MFP: Optional
    - https://forum.openwrt.org/t/i-need-help-how-to-prevent-wifi-from-dissociation-attack/85229

### Static leases

- **Network** --> **DHCP and DNS** --> **Static Leases**
- chronos (Synology NAS) -> 192.168.1.100
- Dahua XVR -> 192.168.1.101
- vulcan (RPi backup NAS) -> 192.168.1.102

### syslog

- syslog server is running on [NAS](https://github.com/gemesa/nexus-net/blob/main/nas.md?plain=1#L181)
- TLDR
  ```
  $ nano /etc/config/system
  $ cat /etc/config/system
  config system
  ...
    option log_ip '192.168.1.100'
    option log_port '514'
    option log_proto 'tcp'
  $ /etc/init.d/log restart
  ```
- references
  - https://openwrt.org/docs/guide-user/base-system/log.essentials#network_logging
  - https://forum.openwrt.org/t/solved-openwrt-is-not-sending-syslog-messages-to-external-syslog-server/77078/4

### DNS

- configure DoH
  - TLDR
    - install `https-dns-proxy`
    - install `luci-app-https-dns-proxy` (**Services** --> **HTTPS DNS Proxy**)
  - references
    - https://openwrt.org/docs/guide-user/services/dns/doh_dnsmasq_https-dns-proxy#command-line_instructions
    - https://openwrt.org/docs/guide-user/services/dns/doh_dnsmasq_https-dns-proxy#web_interface
- *optional:* enforce DNS hijacking on all interfaces not just `lan` (which is already handled by `https-dns-proxy`)
  - https://openwrt.org/docs/guide-user/firewall/fw3_configurations/intercept_dns
  - https://forum.openwrt.org/t/does-https-dns-proxy-protect-against-dns-hijacking/107602/3
  - https://forum.openwrt.org/t/question-on-dns-hijacking/149113

### DDNS

- https://www.dynu.com/en-US/
  - **Control Panel** --> **DDNS Services**
    - set initial IPv4 manually via http://checkip.dyndns.com
- TLDR
  - install `ddns-scripts`
  - install `luci-app-ddns`
  - **Services** --> **Dynamic DNS** --> Edit `myddns_ipv4`
    - **Basic Settings**
      - enable
      - Lookup Hostname (same as Domain)
      - DDNS Service provider: dynu.com
      - Domain (same as Lookup Hostname)
      - Username
      - Password (use IP Update Password instead of account password)
      - Use HTTP Secure
      - Path to CA-Certificate: /etc/ssl/certs/ca-certificates.crt
    - **Advanced Settings**
      - IP address source: URL
      - URL to detect: http://checkip.dyndns.com
  - **Services** --> **Dynamic DNS** --> Reload `myddns_ipv4`
- references
  - https://openwrt.org/docs/guide-user/services/ddns/client
  - https://www.youtube.com/watch?v=OWZkjawcM8A

### Tailscale

#### `tailscale` package

For devices with enough disk space.

```
apk add tailscale
```

#### `openwrt-tailscale-enabler`

For devices with enough RAM.

https://github.com/adyanth/openwrt-tailscale-enabler

#### Tailscale static binaries

For devices with unsufficient disk space and RAM (use external USB storage).

```
# Install USB/filesystem packages
apk add kmod-usb-storage kmod-fs-ext4 block-mount e2fsprogs

# Format and mount the USB drive
mkfs.ext4 -F /dev/sda1
mkdir -p /mnt/usb
mount /dev/sda1 /mnt/usb

# Configure fstab for auto-mount on boot
block detect
# copy the 'mount' section to /etc/config/fstab
cat /etc/config/fstab
config 'global'
	option	anon_swap	'0'
	option	anon_mount	'0'
	option	auto_swap	'1'
	option	auto_mount	'1'
	option	delay_root	'5'
	option	check_fs	'0'

config 'mount'
	option	target	'/mnt/usb'
	option	uuid	'YOUR-UUID-HERE'
	option	enabled	'1'
mount -a
mount | grep /mnt/usb

# Download and extract Tailscale binaries onto the USB drive (little-endian)
# for big-endian binaries, see below
mkdir -p /mnt/usb/tailscale-bin
cd /mnt/usb/tailscale-bin
# Check arch
uname -a
wget https://pkgs.tailscale.com/stable/tailscale_1.102.3_mips.tgz
tar xzf tailscale_1.102.3_mips.tgz
mv tailscale_1.102.3_mips/tailscale .
mv tailscale_1.102.3_mips/tailscaled .
rm -rf tailscale_1.102.3_mips tailscale_1.102.3_mips.tgz
chmod +x tailscale tailscaled

# big-endian binaries
# Fedora host
sudo dnf install golang
git clone https://github.com/tailscale/tailscale.git
git fetch --tags
git checkout v1.102.3
GOTOOLCHAIN=auto GOOS=linux GOARCH=mipsle GOMIPS=softfloat go build -o tailscale ./cmd/tailscale
GOTOOLCHAIN=auto GOOS=linux GOARCH=mipsle GOMIPS=softfloat go build -o tailscaled ./cmd/tailscaled
scp -i ~/.ssh/id_ed25519_helios_np tailscale tailscaled root@192.168.1.1:/mnt/usb/tailscale-bin/

# Symlink into /usr/sbin
ln -s /mnt/usb/tailscale-bin/tailscale /usr/sbin/tailscale
ln -s /mnt/usb/tailscale-bin/tailscaled /usr/sbin/tailscaled

# /usr/bin/tailscaled is hardcoded in /etc/init.d/tailscale
ln -sf /mnt/usb/tailscale-bin/tailscaled /usr/bin/tailscaled

# iptables, ip6tables, kmod-tun packages are needed
# linuxfw: clear iptables: exec: "iptables": executable file not found in $PATH
# linuxfw: clear ip6tables: exec: "ip6tables": executable file not found in $PATH
# Missing required package kmod-tun; run: apk add kmod-tun
apk add iptables ip6tables kmod-tun

# Create the init.d service script
# https://github.com/adyanth/openwrt-tailscale-enabler/blob/main/etc/init.d/tailscale
cat /etc/init.d/tailscale
#!/bin/sh /etc/rc.common

# Copyright 2020 Google LLC.
# SPDX-License-Identifier: Apache-2.0

USE_PROCD=1
START=99
STOP=1

start_service() {
  procd_open_instance
  procd_set_param command /usr/bin/tailscaled

  # Set the port to listen on for incoming VPN packets.
  # Remote nodes will automatically be informed about the new port number,
  # but you might want to configure this in order to set external firewall
  # settings.
  procd_append_param command --port 41641

  # OpenWRT /var is a symlink to /tmp, so write persistent state elsewhere.
  procd_append_param command --state /etc/config/tailscaled.state
  
  # Persist files for TLS cert & Taildrop files
  procd_append_param command --statedir /etc/tailscale/

  procd_set_param respawn
  procd_set_param stdout 1
  procd_set_param stderr 1

  procd_close_instance
}

stop_service() {
  /usr/bin/tailscaled --cleanup
}
chmod +x /etc/init.d/tailscale
mkdir -p /etc/tailscale

# Enable, start and authenticate
/etc/init.d/tailscale enable
/etc/init.d/tailscale start
tailscale up --accept-dns=false --advertise-routes=192.168.10.0/24,192.168.0.0/24

# Add tailscale0 to the LAN firewall zone (trust tailnet traffic like LAN)
uci show firewall | grep "zone.*name='lan'"   # confirm the zone index, e.g. @zone[0]
uci add_list firewall.@zone[0].device='tailscale0'
# or alternatively (if using named sections)
uci add_list firewall.lan.device='tailscale0'
uci commit firewall
service firewall restart

# Verify
tailscale status
tailscale ip -4
```

### WireGuard server

- configure port forwarding (51820 - UDP) when behind a NAT

#### Install WG and add peers

- install `wireguard-tools`
- install `luci-proto-wireguard`
- follow https://openwrt.org/docs/guide-user/services/vpn/wireguard/server
  - when adding peers:
    - generate new keys for each peer
    - add description
      `uci set network.wgclient.description="peer0"`
    - route allowed IPs
      `uci set network.wgclient.route_allowed_ips="1"`
    - increment allowed IPs
      `uci add_list network.wgclient.allowed_ips="${VPN_ADDR%.*}.2/32"`
      `uci add_list network.wgclient.allowed_ips="${VPN_ADDR6%:*}:2/128"`

#### Create client configs

- TLDR example
  ```
  [Interface]
  Address = 192.168.9.4/32, fdf1:e8a1:8d3f:9::4/128
  PrivateKey = <content of wgclient.key>
  DNS = 192.168.9.1
  [Peer]
  PublicKey = <content of wgserver.pub>
  PresharedKey = <content of wgclient.psk>
  AllowedIPs = 0.0.0.0/0, ::/0
  Endpoint = <DDNS hostname>:51820
  ```

- references
  - https://serverfault.com/questions/1058255/configure-dns-routing-in-wireguard
  - https://www.reddit.com/r/WireGuard/comments/eeaysn/creating_config_file_for_windows_clients_to_import/
  - https://upcloud.com/community/tutorials/get-started-wireguard-vpn/
  - set DNS server on client side to avoid DNS leak
    - https://forum.openwrt.org/t/solved-wireguard-dns-leak-on-the-mobile-device/14640/3
  - configure full tunnel
    - server config: `route allowed ips`
    - client config: `allowed ips: 0.0.0.0/0, ::/0`
    - https://www.reddit.com/r/WireGuard/comments/k7yb0f/need_help_split_tunnel_works_full_tunnel_doesnt/
  - generate a pre-shared key to add an additional layer of symmetric-key cryptography to be mixed into the already existing public-key cryptography, for post-quantum resistance
    - https://wiki.archlinux.org/title/WireGuard

### WOL

- install `etherwake`
- usage: `etherwake -i br-lan XX:XX:XX:XX:XX:XX`

### Zones

- TLDR
  - **Network**
    - --> **Firewall**
      - --> **General Settings**
        - add zone `Guest`
          - Allow forward to destination zones: add `wan`
        - add zone `IOT`
        - edit `lan`
          - Allow forward to destination zones: add `IOT`
      - --> **Traffic Rules**
        - add rule
          - Name: Guest DHCP and DNS
          - Protocol: TCP and UDP
          - Source zone: Guest
          - Destination port: 53 67 68
    - --> **Interfaces**
      - add interface
        - Name: guest
        - Protocol: Static address
        - IPv4 Address: 10.20.30.40
        - IPv4 netmask: 255.255.255.0
        - **DHCP Server** tab --> Set Up DHCP Server
        - **Firewall Settings** tab --> Create / Assign firewall-zone: Guest
      - add interface
        - Name: iot
        - IPv4 Address: 172.16.0.1
        - **Firewall Settings** tab --> Create / Assign firewall-zone: IOT
        - other settings are the same as above
    - --> **Wireless**
      - add interface
        - ESSID: Helios-Guest
        - Network: `guest`
        - refer to [Wireless](#wireless) for the remaining settings
      - add interface
        - ESSID: Helios-IOT
        - Network: `iot`
        - refer to [Wireless](#wireless) for the remaining settings
  - XVR
    - change the IP to 172.16.0.2
    - remove the `Dahua XVR -> 192.168.1.101` static lease
- references
  - https://www.youtube.com/watch?v=UvniZs8q3eU

### System hardening

- *optional*: https://openwrt.org/docs/guide-user/security/secure.access
