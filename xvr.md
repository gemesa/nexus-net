## Installation and configuration

### General info

- stock firmware
- access: http://192.168.1.101 (Edge in IE mode)

### Config

#### Camera

![camera-channel-type](xvr/camera-channel-type.PNG)

![camera-encode-encode](xvr/camera-encode-encode.PNG)

![camera-encode-overlay](xvr/camera-encode-overlay.PNG)

![camera-encode-snapshot](xvr/camera-encode-snapshot.PNG)

![camera-image](xvr/camera-image.PNG)

![camera-ptz](xvr/camera-ptz.PNG)

#### Network

![network-tcp-ip](xvr/network-tcp-ip.PNG)

![network-connection](xvr/network-connection.PNG)

- disable everything else:
  - Wi-Fi
  - 3G/4G
  - PPPoE
  - DDNS
  - EMAIL
  - UPnP
  - SNMP
  - MULTICAST

#### Storage

- SFTP is not working for some reason --> use FTP
- https://dahuawiki.com/FTP/FTP_Setup_Core_FTP_Server
- https://dahuawiki.com/FTP/FTP_Snapshot_Setup

![storage-basic](xvr/storage-basic.PNG)

![storage-ftp](xvr/storage-ftp.PNG)

![storage-record](xvr/storage-record.PNG)

![storage-schedule-record](xvr/storage-schedule-record.PNG)

![storage-schedule-snapshot](xvr/storage-schedule-snapshot.PNG)

#### System

![system-default](xvr/system-default.PNG)

![system-general-general](xvr/system-general-general.PNG)

![system-general-date-time](xvr/system-general-date-time.PNG)

![system-imp-exp](xvr/system-imp-exp.PNG)

note: export configuration to file

![system-security-firewall](xvr/system-security-firewall.PNG)

![system-security-system-service](xvr/system-security-system-service.PNG)

![system-system-maintain](xvr/system-system-maintain.PNG)

![system-upgrade-online-upgrade](xvr/system-upgrade-online-upgrade.PNG)

### Other

#### Convert .dav to .mp4

- `ffmpeg -y -i video.dav -vcodec libx265 -movflags +faststart video.mp4`
- https://ipcamtalk.com/threads/recommend-dav-to-mp4-converter-please.30193/
- https://trac.ffmpeg.org/wiki/Encode/H.265
- https://ffmpeg.org/download.html#build-windows
- https://www.gyan.dev/ffmpeg/builds/
