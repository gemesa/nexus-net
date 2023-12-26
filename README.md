# nexus-net

This repo describes my home network.

## Network topology


```
                      +-----------+
                      | Internet  |
                      +-----------+
                            |
                     +-------------+
                     | ISP Router  |
                     +-------------+
                            |
                       +---------+
                       | Router  |
                       +---------+
                            |
             +-----------------------------+
             |                             |
   +-------------------+         +-------------------+
   |      Switch       |         |      Switch       |
   +-------------------+         +-------------------+
    |       |         |               |         | 
 +-----+ +-----+ +---------+       +-----+   +-----+           
 | PC  | | RPi | | IP Cam  |       | NAS |   | XVR |           
 +-----+ +-----+ +---------+       +-----+   +-----+           
                                                |
                               +-------------+  |  +-------------+
                               | HDCVI Cam 0 |--+--| HDCVI Cam 1 |
                               +-------------+  |  +-------------+
                                                |
                               +-------------+  |  +-------------+
                               | HDCVI Cam 2 |--+--| HDCVI Cam 3 |
                               +-------------+  |  +-------------+
                                                |
                               +-------------+  |  +-------------+
                               | HDCVI Cam 4 |--+--| HDCVI Cam 5 |
                               +-------------+  |  +-------------+
                                                |
                               +-------------+  |  +-------------+
                               | HDCVI Cam 6 |--+--| HDCVI Cam 7 |
                               +-------------+     +-------------+
```

## Devices

### ISP Router

[UPC Connect Box](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwi82ZWt6q2DAxWY_bsIHUgUAiQQFnoECBIQAQ&url=https%3A%2F%2Fwww.upc.ch%2Fpdf%2Fsupport%2Fen%2Fmanuals%2Finternet%2Fconnectbox%2Fconnect-box-manual.pdf&usg=AOvVaw1POAA5CCxkLlS9mlO_BAVz&opi=89978449):
- bridge mode
- can not be accessed via browser
- can be reset (default: router mode), default credentials: at the bottom of the box

### Router

[Netgear Nighthawk X4S R7800](https://www.netgear.com/home/wifi/routers/r7800/):
- hostname: helios
- router mode
- [OpenWrt firmware](https://openwrt.org/toh/netgear/r7800)
- can be accessed via browser (http://helios.lan or http://192.168.1.1)
- can be reset (default: router mode), no password necessary after reset
- can be accessed via ssh (key authentication)
- hosts [WireGuard VPN server](https://openwrt.org/docs/guide-user/services/vpn/wireguard/server)

### Switch

[TP-Link TL-SG105](https://www.tp-link.com/hu/business-networking/unmanaged-switch/tl-sg105/):
- unmanaged switch
- no PoE

### NAS

[Synology DS920+](https://global.download.synology.com/download/Document/Hardware/DataSheet/DiskStation/20-year/DS920+/enu/Synology_DS920_Plus_Data_Sheet_enu.pdf):
- 4x4TB [Seagate Ironwolf HDDs](https://www.seagate.com/gb/en/products/nas-drives/ironwolf-hard-drive/)
- UPS: [APC Smart-UPS 750VA SmartConnect](https://www.apc.com/shop/hr/en/products/APC-Smart-UPS-Line-Interactive-750VA-Tower-230V-6x-IEC-C13-outlets-SmartConnect-Port-SmartSlot-AVR-LCD/P-SMT750IC)
- raid type: SHR (with data protection for 1-drive fault tolerance)
- file system: Btrfs
