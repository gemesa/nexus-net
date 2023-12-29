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
- install download station at **Package Center** --> **Download Station**
  - create a "P2P" shared folder (without recycle bin and checksum)
  - create a dedicated P2P user which can only access this shared folder (this account will be used for accessing **Download Station** and the downloaded files)

### Surveillance

- enable FTP service at **Control Panel** --> **File Services** --> **FTP**
  - create a "CCTV" shared folder (without recycle bin and checksum)
  - create a dedicated FTP user which can only access this shared folder (XVR will log in and sync CCTV data using this account)
- install surveillance station at **Package Center** --> **Surveillance Station**
  - integrate IP cam(s) using ONVIF

### Scheduled tasks

- schedule tasks at **Control Panel** --> **Task Scheduler**

#### Task 0: empty recycle bins

- user: root
- schedule: repeat daily
- action: empty all recycle bins
- retention policy: number of days to retain deleted files: 14

#### Task 1: send email reminders

- create a "mail" shared folder (with encryption, recycle bin and checksum)
- create a dedicated mail user which can only access this shared folder
  - this account will be used to send automated email reminders
  - this folder will contain the sendmail script and the email messages
- user: dedicated mail user
- schedule: custom
- notification: send run details by email
- run command: `bash /volume1/mail/send-mail.sh`

```
$ cat send-mail.sh
#!/bin/bash
/sbin/sendmail -v -t < /volume1/mail/msg.txt
```

#### Task 2: remove old CCTV files

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
- TODO: review https://www.itsmdaily.com/how-to-secure-synology-nas-against-exploits-malware-cryptolockers/
- TODO: review https://www.youtube.com/watch?v=916idkMTuKg

### DNS

- hostname: chronos
