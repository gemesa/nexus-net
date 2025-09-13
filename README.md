# nexus-net

This repository serves as a snapshot of my personal home network setup. Passwords and private keys are securely stored in a password manager.

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
                                     +---------+         +----+
                                     | Router  |   ~~~   | RE |
                                     +---------+         +----+
                                          |
                      +---------------------------------------+
                      |                                       |
  +----+   +---------------------+        +----+   +---------------------+   +----+
  | PC |---|       Switch        |        | PC |---|       Switch        |---| PC |
  +----+   +---------------------+        +----+   +---------------------+   +----+
            |         |         |                        |         |
         +-----+ +---------+ +-----+   +-------+      +-----+   +-----+   +-------+
         | PC  | | IP Cam  | | RPi |---| UPS 0 |      | XVR |   | NAS |---| UPS 1 |
         +-----+ +---------+ +-----+   +-------+      +-----+   +-----+   +-------+
                                |                        |
                             +-----+    +-------------+  |  +-------------+
                             | EHD |    | HDCVI Cam 0 |--+--| HDCVI Cam 1 |
                             +-----+    +-------------+  |  +-------------+
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

### ISP Router - [UPC Connect Box](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwi82ZWt6q2DAxWY_bsIHUgUAiQQFnoECBIQAQ&url=https%3A%2F%2Fwww.upc.ch%2Fpdf%2Fsupport%2Fen%2Fmanuals%2Finternet%2Fconnectbox%2Fconnect-box-manual.pdf&usg=AOvVaw1POAA5CCxkLlS9mlO_BAVz&opi=89978449)

#### Overview

- bridge mode

#### Details

See [isp-router.md](isp-router.md)

---

### Router - [ASUS RT-AX53U](https://openwrt.org/toh/asus/rt-ax53u)

#### Overview

- router mode
- OpenWrt firmware
- access:
  - http://helios.lan or http://192.168.1.1
  - ssh (key auth enabled, pw auth disabled)
- WireGuard server (with DDNS)
- encryption: mixed WPA2/WPA3 PSK, SAE (CCMP)

#### Details

See [router.md](router.md)

---

### RE - [TP-Link RE315](https://www.tp-link.com/us/home-networking/range-extender/re315/)

#### Overview

- 2.4GHz only (5GHz disabled)
- encryption: WPA2/WPA3-Personal

---

### Switch - [TP-Link TL-SG105](https://www.tp-link.com/hu/business-networking/unmanaged-switch/tl-sg105/)

#### Overview

- unmanaged switch
- no PoE

---

### NAS - [Synology DS920+](https://global.download.synology.com/download/Document/Hardware/DataSheet/DiskStation/20-year/DS920+/enu/Synology_DS920_Plus_Data_Sheet_enu.pdf)

#### Overview

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

---

### UPS 1 - [APC Smart-UPS 750VA SmartConnect](https://www.apc.com/shop/hr/en/products/APC-Smart-UPS-Line-Interactive-750VA-Tower-230V-6x-IEC-C13-outlets-SmartConnect-Port-SmartSlot-AVR-LCD/P-SMT750IC)

#### Overview

- connected to NAS via USB

---

### XVR - [DH-XVR5108HS-X](https://www.dahuasecurity.com/asset/upload/uploads/soft/20200529/XVR5108-16HS-X_Datasheet_20200529.pdf)

#### Overview

- access: http://192.168.1.101 (Edge in IE mode)
- syncs CCTV data via FTP to NAS

#### Details

See [xvr.md](xvr.md)

---

### HDCVI Camera - [DH-HAC-HFW1200D-0360B-S4](https://www.dahuasecurity.com/asset/upload/download/DH-HAC-HFW1200D_Datasheet_20171127.pdf)

#### Overview

- 1080P
- night vision

---

### IP Camera - [D-Link DCS-8526LH](https://www.dlink.com/en/products/dcs-8526lh-mydlink-full-hd-pan--tilt-pro-wi-fi-camera)

#### Overview

- 1080P
- night vision
- connected to NAS **Surveillance Station** via ONVIF

---

### PC - [Lenovo ThinkPad L580](https://www.lenovo.com/us/en/p/laptops/thinkpad/thinkpadl/thinkpad-l580/22tp2tbl580)

#### Overview

- Fedora

#### Details

See [pc-thinkpad-l580.md](pc-thinkpad-l580.md)

---

### PC - [MINISFORUM X500 5700G](https://www.amazon.com/MINISFORUM-Elitemini-X500-Computer-Graphics/dp/B09KTBG275)

#### Overview

- Fedora

#### Details

See [pc-minisforum-x500-5700g.md](pc-minisforum-x500-5700g.md)

---

### PC - [Mac Mini M1](https://support.apple.com/en-us/111894)

#### Overview

- macOS 15

#### Details

See [pc-mac-mini-m1.md](pc-mac-mini-m1.md)

---

### PC - [Radxa ROCK 5C](https://radxa.com/products/rock5/5c/)

#### Overview

- [Armbian](https://www.armbian.com/)

#### Details

See [pc-radxa-rock-5c.md](pc-radxa-rock-5c.md)

---

### Phone - [Galaxy S10e 128 GB](https://www.samsung.com/hu/support/model/SM-G970FZWDXEH)

#### Overview

- Stock firmware

#### Details

See [phone-galaxy-s10e.md](phone-galaxy-s10e.md)

---

### RPi - [Raspberry Pi 4 model B 4GB](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/specifications/)

#### Overview

- [Revolt Pi 4 Cool Box](https://malnapc.hu/revolt-pi-4-cool-box-grey?keyword=Revolt%20Pi%204%20Cool%20Box%20-%20Grey)
- 32GB SD card
- encrypted with [cryptmypi](https://github.com/unixabg/cryptmypi)
  - image: [Buster](https://downloads.raspberrypi.com/raspios_lite_armhf/images/raspios_lite_armhf-2021-05-28)
- connected to EHD via USB
- backup file server (syncs data from NAS via ssh)
- access: ssh (key auth enabled, pw auth disabled)
- mail server (SMTP relay)

#### Details

See [rpi.md](rpi.md)

---

### UPS 0 - [Eaton 5SC500I](https://www.eaton.com/hu/hu-hu/skuPage.5SC500I.html)

#### Overview

- TODO: connect the RPi to the currently unused USB port

---

### EHD - [WD Elements Desktop Hard Drive](https://www.westerndigital.com/en-ap/products/external-drives/wd-elements-desktop-usb-3-0-hdd?sku=WDBBKG0020HBK-SESN)

#### Overview

- 4TB
