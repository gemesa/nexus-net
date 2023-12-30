# nexus-net

This repository is a snapshot of my home network setup.

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
 +-----+ +-----+ +---------+       +-----+   +-----+   +-----+
 | PC  | | RPi | | IP Cam  |       | XVR |   | NAS |---| UPS |
 +-----+ +-----+ +---------+       +-----+   +-----+   +-----+
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

#### Overview

- [UPC Connect Box](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwi82ZWt6q2DAxWY_bsIHUgUAiQQFnoECBIQAQ&url=https%3A%2F%2Fwww.upc.ch%2Fpdf%2Fsupport%2Fen%2Fmanuals%2Finternet%2Fconnectbox%2Fconnect-box-manual.pdf&usg=AOvVaw1POAA5CCxkLlS9mlO_BAVz&opi=89978449)
- bridge mode
- no access (no http and no ssh)

#### Details

See [isp-router.md](isp-router.md)

### Router

#### Overview

- [Netgear Nighthawk X4S R7800](https://www.netgear.com/home/wifi/routers/r7800/)
- router mode
- OpenWrt firmware
- access:
  - http://helios.lan or http://192.168.1.1
  - ssh (key auth enabled, pw auth disabled)
- WireGuard server

#### Details

See [router.md](router.md)

### Switch

#### Overview

- [TP-Link TL-SG105](https://www.tp-link.com/hu/business-networking/unmanaged-switch/tl-sg105/)
- unmanaged switch
- no PoE

### NAS

#### Overview

- [Synology DS920+](https://global.download.synology.com/download/Document/Hardware/DataSheet/DiskStation/20-year/DS920+/enu/Synology_DS920_Plus_Data_Sheet_enu.pdf)
- access:
  - http://chronos.lan or http://192.168.1.100
  - ssh (both key and pw auth enabled)
- file server (SMB, FTP)
  - XVR syncs CCTV data via FTP
- syslog server
- P2P file sharing server
- surveillance server (IP cams)
- mail server (SMTP relay)

#### Details

See [nas.md](nas.md)

### UPS

#### Overview

- [APC Smart-UPS 750VA SmartConnect](https://www.apc.com/shop/hr/en/products/APC-Smart-UPS-Line-Interactive-750VA-Tower-230V-6x-IEC-C13-outlets-SmartConnect-Port-SmartSlot-AVR-LCD/P-SMT750IC)
- connected to NAS via USB

### XVR

#### Overview

- [XVR5108HS-X](https://www.dahuasecurity.com/asset/upload/uploads/soft/20200529/XVR5108-16HS-X_Datasheet_20200529.pdf)
- access: http://192.168.1.101 (Edge in IE mode)
- syncs CCTV data via FTP to NAS

#### Details

See [xvr.md](xvr.md)

### HDCVI Camera

#### Overview

- TODO: add product link

### IP Camera

#### Overview

- [D-Link DCS-8526LH](https://www.dlink.com/en/products/dcs-8526lh-mydlink-full-hd-pan--tilt-pro-wi-fi-camera)
- connected to NAS **Surveillance Station** via ONVIF

### PC

#### Overview

- [Lenovo ThinkPad L580](https://www.lenovo.com/us/en/p/laptops/thinkpad/thinkpadl/thinkpad-l580/22tp2tbl580)
- Fedora
