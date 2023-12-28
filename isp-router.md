## Installation and configuration

### General info

- stock firmware
- after reset:
  - default mode: router
  - default credentials: at the bottom of the box

### Bridge mode

- bridge mode can be set either manually or by calling the ISP
- no access (no http and no ssh)

### Router mode

- if the device can not be set to bridge mode for some reason, we need to use it in router mode (double NAT)
- disable wireless (2.4G, 5G and guest)
- disable guest network
- disable remote access
- disable UPnP
- disable DMZ
- disable WPS
- disable port forwarding and delete existing rules
  - exception: OpenVPN 1194 / WireGuard 51820
- enable DHCP
