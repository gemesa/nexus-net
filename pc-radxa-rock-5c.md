## Installation and configuration

### Install

- https://armbian.com/boards/rock-5c
- https://docs.radxa.com/en/rock5/rock5c/getting-started/install-os/boot_from_emmc
- https://docs.radxa.com/en/rock5/rock5c/other-os/armbian
- https://github.com/radxa-pkg/aic8800/releases

```
$ sudo apt remove aic8800-usb-dkms
# install latest release (https://github.com/radxa-pkg/aic8800/releases)
$ wget https://github.com/radxa-pkg/aic8800/releases/download/5.0%2Bgit20260123.5f7be68d-5/aic8800-firmware_5.0+git20260123.5f7be68d-5_all.deb
$ wget https://github.com/radxa-pkg/aic8800/releases/download/5.0%2Bgit20260123.5f7be68d-5/aic8800-usb-dkms_5.0+git20260123.5f7be68d-5_all.deb
$ sha256sum *.deb
$ sudo apt install ./aic8800-firmware_5.0+git20260123.5f7be68d-5_all.deb ./aic8800-usb-dkms_5.0+git20260123.5f7be68d-5_all.deb
$ sudo apt update && sudo apt upgrade -y
```

#### Errors during upgrade

```
$ sudo apt update && sudo apt upgrade -y
...
Errors were encountered while processing:
 linux-image-vendor-rk35xx
Error: Sub-process /usr/bin/dpkg returned an error code (1)
```

Solution: run `$ sudo apt update && sudo apt upgrade -y` again.

### Setup

#### ssh

- change root pw (default: 1234)
- create non-root user
