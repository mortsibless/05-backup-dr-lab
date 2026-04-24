[GitHub_05_Backup_DR_README.md](https://github.com/user-attachments/files/27048271/GitHub_05_Backup_DR_README.md)
# 💾 Backup & Disaster Recovery Lab

![Windows Server](https://img.shields.io/badge/Windows_Server-2019-0078D6?style=flat-square&logo=windows&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Bash](https://img.shields.io/badge/Scripting-Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-blue?style=flat-square)

A practical lab focused on designing, implementing, and testing backup strategies and disaster recovery procedures across Windows and Linux environments. Built around the principle that a backup strategy is only as good as the last successful restore test.

---

## 🎯 Objective

To demonstrate the ability to implement and verify backup and recovery solutions — a critical skill for any SysAdmin responsible for ensuring business continuity and minimising data loss in the event of system failure.

---

## 🛠️ Environment

| Component | Details |
|---|---|
| Windows Backup Target | Windows Server 2019 (DC-01) |
| Linux Backup Target | Ubuntu Server 22.04 (WEB-01) |
| Backup Destination | Separate virtual disk + external simulation |
| Tools Used | Windows Server Backup, rsync, cron, wbadmin |

---

## ✅ What Was Built & Practised

### 1. Windows Server Backup (Full System Image)

**Setup:**
- Installed Windows Server Backup feature via Server Manager
- Configured scheduled daily backup to a dedicated virtual disk
- Verified backup completion via Event Viewer (Event ID 4)

**Backup command (wbadmin):**
```powershell
# Trigger manual backup via command line
wbadmin start backup -backuptarget:E: -include:C: -allCritical -quiet

# Check backup status
wbadmin get status

# List available backups
wbadmin get versions
```

**Recovery test:**
- Booted from Windows Server 2019 installation media
- Selected "Repair your computer" → System Image Recovery
- Successfully restored full system image and verified domain services came back online

### 2. Linux Automated Backup with rsync & Cron

**Backup script:**
```bash
#!/bin/bash
# linux_backup.sh — Incremental backup with logging

SOURCE="/home /etc /var/www"
DEST="/mnt/backup/$(date '+%Y-%m-%d')"
LOGFILE="/var/log/backup.log"

mkdir -p "$DEST"
echo "[$(date)] Starting backup to $DEST" >> $LOGFILE

rsync -avz --delete $SOURCE $DEST >> $LOGFILE 2>&1

if [ $? -eq 0 ]; then
    echo "[$(date)] Backup completed successfully." >> $LOGFILE
else
    echo "[$(date)] ERROR: Backup failed." >> $LOGFILE
fi
```

**Cron schedule — daily at 2:00 AM:**
```bash
0 2 * * * /home/mortsi/scripts/linux_backup.sh
```

**Backup rotation — keep last 7 days:**
```bash
find /mnt/backup/ -maxdepth 1 -type d -mtime +7 -exec rm -rf {} \;
```

### 3. RTO & RPO Planning

| Term | Definition | Lab Target |
|---|---|---|
| **RPO** (Recovery Point Objective) | Max acceptable data loss | 24 hours (daily backup) |
| **RTO** (Recovery Time Objective) | Max acceptable downtime | 2 hours (system image restore) |

Documented a basic Business Continuity checklist:
- [ ] Identify critical services (AD, DNS, DHCP, file shares)
- [ ] Define backup frequency per service tier
- [ ] Test restore procedure quarterly
- [ ] Store one backup copy off-site (simulated with separate virtual disk)
- [ ] Document recovery runbook and keep it accessible offline

### 4. Disaster Recovery Scenarios Tested

| Scenario | Backup Type Used | Outcome |
|---|---|---|
| Accidental file deletion | rsync incremental | Files restored from previous day's backup ✅ |
| OS corruption (Windows) | Full system image | Domain Controller restored and functional ✅ |
| Web server data loss | rsync + cron | `/var/www` restored, site back online in 8 mins ✅ |
| Ransomware simulation | Isolated VM + snapshot | Rolled back VM to clean snapshot ✅ |

---

## 📝 Key Lessons Learned

1. **Always test restores** — A backup you haven't restored from is just a file you haven't read
2. **Automate and log everything** — Silent failures are worse than no backup at all
3. **Separate backup destination** — Backup on the same disk as the source is not a backup
4. **Document the recovery process** — In a real incident, you won't have time to figure it out

---

## 📸 Screenshots

> _Windows Server Backup console, rsync log output, and recovery boot screenshots

---

## 📚 Skills Demonstrated

`Windows Server Backup` `wbadmin` `rsync` `Bash Scripting` `cron` `Disaster Recovery` `RTO/RPO Planning` `System Image Restore` `Backup Automation` `Business Continuity` `Log Monitoring`

---

## 🔗 Related Projects

- [Active Directory Infrastructure Lab](../01-active-directory-lab)
- [Linux Server Administration Lab](../03-linux-server-lab)
- [Enterprise Virtual Lab Environment](../04-virtualisation-lab)
