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
- **System** --> **Software**
  - run **Update lists...**
  - install packages
    - `nano`
    - `fdisk`
    - `lsblk`
    - `pciutils`
    - `usbutils`
    - `luci-app-statistics`

## Network

- **Network** --> **Firewall** --> **General Settings**
  - **Drop invalid packets**: enable
  - **Software flow offloading**: enable

### Wireless

- **Network** --> **Wireless**
- SSID: Helios
- SSID: Helios-5G
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

- syslog server is running on NAS
- https://openwrt.org/docs/guide-user/base-system/log.essentials
- https://forum.openwrt.org/t/solved-openwrt-is-not-sending-syslog-messages-to-external-syslog-server/77078

### DNS

- set hostname: helios at **System** --> **System**
  - access: http://helios.lan
- configure encrypted DNS
  - https://openwrt.org/docs/guide-user/services/dns/doh_dnsmasq_https-dns-proxy
  - install related LuCI UI package as well
- configure DNS hijacking
  - https://openwrt.org/docs/guide-user/firewall/fw3_configurations/intercept_dns
  - https://forum.openwrt.org/t/does-https-dns-proxy-protect-against-dns-hijacking/107602/3

### DDNS

- https://www.dynu.com/en-US/
- https://openwrt.org/docs/guide-user/services/ddns/client
- https://www.youtube.com/watch?v=OWZkjawcM8A

### WireGuard server

#### Install WG

- https://www.youtube.com/watch?v=Bo2AsW4BMOo
- https://openwrt.org/docs/guide-user/services/vpn/wireguard/server
- TODO: configure WG logging

#### Create client configs

- https://serverfault.com/questions/1058255/configure-dns-routing-in-wireguard
- https://www.reddit.com/r/WireGuard/comments/eeaysn/creating_config_file_for_windows_clients_to_import/
- https://upcloud.com/community/tutorials/get-started-wireguard-vpn/
- set DNS server on client side to avoid DNS leak
  - https://forum.openwrt.org/t/solved-wireguard-dns-leak-on-the-mobile-device/14640/3
- configure full tunnel
  - server config: `route allowed ips`
  - client config: `allowed ips: 0.0.0.0/0, ::/0`
  - https://www.reddit.com/r/WireGuard/comments/k7yb0f/need_help_split_tunnel_works_full_tunnel_doesnt/
- utilize preshared keys for maximum security
  - https://wiki.archlinux.org/title/WireGuard
- configure ipv6 address

### Security

- TODO: review router access security
  - https://openwrt.org/docs/guide-user/security/secure.access
  - https://news.ycombinator.com/item?id=17648956
- TODO: review CCTV security
  - https://www.een.com/security-camera-system-cyber-best-practices/
  - https://www.ic2cctv.com/wp-content/uploads/2017/04/iC2-12-ways-Cyberattack_V2.pdf
  - https://www.cctvcameraworld.com/secure-security-camera-system-on-internet/
  - https://www.consumer.ftc.gov/articles/0382-using-ip-cameras-safely
  - https://blog.cammy.com/prevent-ip-camera-hacking
  - https://reolink.com/how-to-secure-your-wifi-enabled-home-camera/

### VLAN

- TODO: create VLANs
  - https://www.youtube.com/watch?v=UvniZs8q3eU
  - https://www.youtube.com/watch?v=4t_S2oWsBpE

### Other

- TODO: test WebRTC leak
- TODO: experiment with bridge/transparent firewall
