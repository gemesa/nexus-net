## Installation and configuration

### Install OpenWrt firmware

#### Netgear Nighthawk X4S R7800

https://openwrt.org/toh/netgear/r7800

#### ASUS RT-AX53U

https://shadowshell.io/flash-openwrt-on-your-asus-rt-ax53u-router

#### TP-Link Archer C7

https://shadowshell.io/unbrick-your-tp-link-archer-c7-openwrt-router

### Generic configuration

- access: http://192.168.1.1
- **System** --> **Administration**
  - --> **Router Password**: set root pw
  - --> **HTTP(S) Access**
    - **Redirect to HTTPS**: enable
  - --> **SSH Access**
    - **Interface**: LAN
    - **Password authentication**: disable
  - --> **SSH-Keys**
    - generate ssh keys and add public key
    - `ssh-keygen -t ed25519`
  - --> **System**
    - **Hostname**: helios (access: https://helios.lan)
    - **Timezone**
  - **Software**
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
- *experimental:* enable 802.11w MFP (some clients may not support this)
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
    option log_ip <destination IP>
    option log_port <destination port>
    option log_proto <tcp or udp>
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
- TLDR
  - install `ddns-scripts`
  - install `luci-app-ddns`
  - **Services** --> **Dynamic DNS** --> Edit `myddns_ipv4`
    - **Basic Settings**
      - enable
      - Lookup Hostname
      - DDNS Service provider: dynu.com
      - Domain
      - Username
      - Password (use IP Update Password instead of account password)
    - **Advanced Settings**
      - IP address source: URL
      - URL to detect: http://checkip.dyndns.com
  - **Services** --> **Dynamic DNS** --> Reload `myddns_ipv4`
- references
  - https://openwrt.org/docs/guide-user/services/ddns/client
  - https://www.youtube.com/watch?v=OWZkjawcM8A

### WireGuard server

- configure port forwarding (51820) when behind a NAT

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
