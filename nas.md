## Installation and configuration

### Generic info/configuration

- access:
  - http://chronos.lan or http://192.168.1.100
- enable ssh at **Control Panel** --> **Terminal & SNMP** --> **Terminal**
- enable email notifications at **Control Panel** --> **Notification** --> **Email**
  - use a dedicated gmail account
  - add personal email to the recipients (rule: warning)
  - change subject prefix from [\<ip>] to [\<hostname>]

### Storage, data and health

- 4x4TB [Seagate Ironwolf HDDs](https://www.seagate.com/gb/en/products/nas-drives/ironwolf-hard-drive/)
- raid type: SHR (1-drive fault tolerance)
- file system: Btrfs
- enable drive health report at **Storage Manager** --> **HDD/SSD** --> **Settings** --> **Advanced**
- enable active insight at **Package Center** --> **Active Insight**
- create a separate shared folder for each user at **Control Panel** --> **Shared Folder**
  - enable encryption
  - enable recycle bin
  - enable checksum
- enable SMB at **Control Panel** --> **File Services** --> **SMB**
- enable rsync service at **Control Panel** --> **File Services** --> **rsync**
  - rsync via ssh is way faster than copying data via SMB when creating backups
  - https://roll.urown.net/NAS/nas_backup.html
  - https://linuxconfig.org/how-to-create-incremental-backups-using-rsync-on-linux
- install download station at **Package Center** --> **Download Station**
  - create a "P2P" shared folder (without recycle bin and checksum)
  - create a dedicated P2P user which can only access this shared folder (this account will be used for accessing **Download Station** and the downloaded files)

### Surveillance

- enable FTP service at **Control Panel** --> **File Services** --> **FTP**
  - create a "CCTV" shared folder (without recycle bin and checksum)
  - create a dedicated FTP user which can only access this shared folder (XVR will log in and sync CCTV data using this account)
- install surveillance station at **Package Center** --> **Surveillance Station**
  - integrate IP cam(s) using ONVIF

#### D-Link DCS-8526LH

- **Surveillance Station** --> **IP Camera** --> **Add**
- network scan does not find the device --> add it manually
  - D-Link
  - DCS-8526LH
  - see pw on bottom of the device
  - install **mydlink** app
    - update firmware
    - set timezone (done automatically)
  - connect through ONVIF with Synology, the camera will be managed by Synology after this and can be removed in **mydlink** app
  - set resolution, fps, bitrate etc

### Scheduled tasks

- schedule tasks at **Control Panel** --> **Task Scheduler**

#### Task 0: empty recycle bins

##### Task config

- user: root
- schedule: repeat daily
- action: empty all recycle bins
- retention policy: number of days to retain deleted files: 14

#### Task 1: send email reminders

##### Shared folder

- create a "mail" shared folder (with encryption, recycle bin and checksum)
- create a dedicated mail user which can only access this shared folder
  - this account will be used to send automated email reminders
  - this folder will contain the sendmail script and the email messages

##### SMTP config

- generate an app pw
  - https://stackoverflow.com/questions/73365098/how-to-turn-on-less-secure-app-access-on-google
- install mail server at **Package Center** --> **Synology Mail Server**
  - https://geektank.net/2022/01/29/sending-email-from-synology-via-cli-ssh-on-dsm-6/
- enable SMTP and SMTP auth at **Synology Mail Server** --> **SMTP**
  - hostname (FQDN): synology.com
- configure SMTP relay at **Synology Mail Server** --> **SMTP** --> **SMTP Relay**
  - enable SMTP relay
  - server: smtp.gmail.com
  - port: 587
  - check "Always use a secure connection (TLS)"
  - check "Authentication required"
    - fill credentials (use the generated app pw)

##### Task config

- user: dedicated mail user
- schedule: custom
- notification: send run details by email
- run command: `bash /volume1/mail/send-mail.sh`

```
$ cat send-mail.sh
#!/bin/bash
/sbin/sendmail -v -t < /volume1/mail/msg.txt
$ cat msg.txt
To: <your-email>
From: <your-email-configured-in-smtp-relay>
Subject: test

<your-message>
```

#### Task 2: remove old CCTV files

##### Task config

- user: dedicated ftp/cctv user
- schedule: repeat daily
- notification: send run details by email
- run command: `bash /volume1/cctv/cleanup.sh`

```
$ cat cleanup.sh
#!/bin/bash

LOG_FILE="/volume1/cctv/cleanup.log"

echo "[START] $(date)" >> $LOG_FILE

find "/volume1/cctv/192.168.1.101" -mtime +60 -type f -exec rm -v "{}" \; >> $LOG_FILE 2>&1 \
&& echo "[SUCCESS - removing files] $(date)" >> $LOG_FILE \
|| echo "[FAIL - removing files] $(date)" >> $LOG_FILE

find "/volume1/cctv/192.168.1.101" -depth -type d -exec rmdir -v "{}" \; >> $LOG_FILE 2>&1 \
&& echo "[SUCCESS - removing folders] $(date)" >> $LOG_FILE \
|| echo "[FAIL - removing folders] $(date)" >> $LOG_FILE

echo "[FINISH] $(date)" >> $LOG_FILE
```

### Hardware & power

- disable hibernation at **Control Panel** --> **Hardware & Power** --> **HDD Hibernation**
  - XVR is syncing data every hour via FTP so hibernation (with its frequent spinning up and down) might do more harm than good
- enable UPS at **Control Panel** --> **Hardware & Power** --> **UPS**
  - UPS type: USB UPS
  - set time before NAS enters standby mode: 20 mins
- enable WOL at **Control Panel** --> **Hardware & Power** --> **General** --> **Power Recovery**
  - disable: Restart automatically when power supply issue is fixed
  - enable: Enable WOL on Lan 1
  - enable: Enable WOL on Lan 2

### Update and backup

- enable auto config backup at **Control Panel** --> **Update & Restore** --> **Configuration Backup**
- enable auto DSM update at **Control Panel** --> **Update & Restore** --> **DSM Update** --> **Update Settings**

### Security

- enable **Security Advisor**
  - change default ssh port to make **Security Advisor** happy
- install AV at **Package Center** --> **Antivirus Essential**
- disable **QuickConnect** at **Control Panel** --> **External Access** --> **QuickConnect**
- set automatic logout timer to 15 mins at **Security** --> **Security**
- enable auto block at **Security** --> **Protection**
- enable account protection at **Security** --> **Account** --> **Account Protection**
  - https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/connection_security_account?version=6
- enable snapshots
  - install snapshot replication at **Package Center** --> **Snapshot Replication**
  - enable snapshots for each shared folder at **Snapshot Replication** --> **Snapshots** --> **Shared Folder** --> **Settings** --> **Schedule**
    - every 6 hours
  - enable retention for each shared folder at **Snapshot Replication** --> **Snapshots** --> **Shared Folder** --> **Settings** --> **Retention**
    - advanced retention policy:
      - keep all snapshots for 1 days
      - keep the latest snapshot for 7 days
      - keep the latest snapshot for 8 weeks
      - keep the latest snapshot for 3 months
      - number of latest snapshots to keep: 5
    - exception: CCTV folder: keep all snapshots for 14 days
- generate and add ssh keys
  - https://samuelsson.dev/log-in-with-ssh-key-authorization-on-a-synology-nas/
  - https://www.youtube.com/watch?v=XN9SuzV08Ew
  - https://kb.synology.com/en-us/DSM/tutorial/How_to_log_in_to_DSM_with_key_pairs_as_admin_or_root_permission_via_SSH_on_computers
  - https://www.reddit.com/r/synology/comments/12drqpz/ssh_key_authentication_on_synology/
- install log center at **Package Center** --> **Log Center**
- enable file station log at **File Station** --> **Settings** --> **General**
- set up log archives at **Log Center** --> **Archive Settings**
  - create a "logs" shared folder (without recycle bin and checksum)
  - select the shared folder as destination
  - check "Archive local logs to the storage location specified above"
  - check "Archive logs as text format in addition to default format"
  - check "Compress log archives"
  - check "Archive logs separately according to device"
  - enable syslog at **Log Center** --> **Log Receiving** --> **Create**
    - name: syslog
    - format: BSD
    - protocol: TCP
    - port: default
  - https://www.youtube.com/watch?v=d5rqwLv1gIU
  - https://kb.synology.com/en-in/DSM/help/LogCenter/logcenter_server?version=6
- TODO: enable 2FA
- TODO: review these:
  - https://www.itsmdaily.com/how-to-secure-synology-nas-against-exploits-malware-cryptolockers/
  - https://www.youtube.com/watch?v=916idkMTuKg
  - https://www.youtube.com/watch?v=G3BJo4B1GgU
  - https://www.youtube.com/watch?v=uausl6HeFjg

### DNS

- hostname: chronos

### Other

#### OpenVPN

- WireGuard is my personal preference over OpenVPN, so these links are here just for archival purposes
- https://www.wundertech.net/synology-nas-openvpn-server-setup-configuration/
- https://community.synology.com/enu/forum/17/post/113967
