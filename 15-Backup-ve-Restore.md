# 🐧 LINUX REHBERİ — BÖLÜM 15: BACKUP VE RESTORE

Yedekleme stratejileri, rsync, tar, automated backups, disaster recovery.

---

## Backup Stratejisi

### 3-2-1 Rule

```
3 copies: Original + 2 backups
2 different media: Disk + Cloud/Tape
1 offsite: Cloud veya remote location
```

---

### Backup Tipleri

| Tip | Açıklama | Süre | Disk | Restore |
|-----|----------|------|------|---------|
| **Full** | Tüm veri | Uzun | Çok | Hızlı |
| **Incremental** | Son backup'tan beri değişen | Kısa | Az | Yavaş |
| **Differential** | Son full'dan beri değişen | Orta | Orta | Orta |

---

## tar - Archive Oluşturma

### Temel Kullanım

```bash
# Oluştur
tar -cvf backup.tar /home/yasin

# Extract
tar -xvf backup.tar

# List
tar -tvf backup.tar

# Gzip ile compress
tar -czvf backup.tar.gz /home/yasin

# Bzip2 (daha iyi compression)
tar -cjvf backup.tar.bz2 /home/yasin

# xz (en iyi compression)
tar -cJvf backup.tar.xz /home/yasin
```

**Flags:**
- `-c`: Create
- `-x`: Extract
- `-t`: List
- `-v`: Verbose
- `-f`: File
- `-z`: gzip
- `-j`: bzip2
- `-J`: xz

---

### Gelişmiş tar

```bash
# Exclude pattern
tar -czvf backup.tar.gz --exclude='*.log' --exclude='node_modules' /home/yasin

# Incremental backup
tar -czf full.tar.gz -g snapshot.file /data
# Sonraki incremental
tar -czf inc1.tar.gz -g snapshot.file /data

# Restore incremental
tar -xzf full.tar.gz -g /dev/null
tar -xzf inc1.tar.gz -g /dev/null

# Yeni dosyaları extract et
tar -xzvf backup.tar.gz --keep-newer-files

# Belirli dosya extract
tar -xzvf backup.tar.gz file.txt

# STDOUT'a extract
tar -xzvf backup.tar.gz -O file.txt

# Progress bar
tar -czf - /data | pv -s $(du -sb /data | awk '{print $1}') > backup.tar.gz

# Remote backup (SSH)
tar -czf - /data | ssh user@remote 'cat > /backup/data.tar.gz'
```

---

## rsync - Synchronization

### Basit Kullanım

```bash
# Lokal sync
rsync -av /source/ /destination/

# Dry run (test)
rsync -av --dry-run /source/ /destination/

# Progress göster
rsync -av --progress /source/ /destination/

# Delete (destination'da olup source'da olmayan sil)
rsync -av --delete /source/ /destination/
```

**Flags:**
- `-a`: Archive mode (recursive, preserve permissions, times, etc.)
- `-v`: Verbose
- `-z`: Compress
- `-h`: Human-readable
- `-P`: Progress + partial (kesintiden devam)
- `--delete`: Extra files sil

---

### Remote Sync (SSH)

```bash
# Local to remote
rsync -avz /local/path/ user@remote:/remote/path/

# Remote to local
rsync -avz user@remote:/remote/path/ /local/path/

# Custom SSH port
rsync -avz -e 'ssh -p 2222' /local/ user@remote:/remote/

# SSH key ile
rsync -avz -e 'ssh -i ~/.ssh/id_rsa' /local/ user@remote:/remote/

# Bandwidth limit (KB/s)
rsync -avz --bwlimit=1000 /local/ user@remote:/remote/
```

---

### Gelişmiş rsync

```bash
# Exclude patterns
rsync -av --exclude='*.log' --exclude='node_modules/' /source/ /dest/

# Exclude from file
rsync -av --exclude-from=exclude.txt /source/ /dest/

# Include/exclude combo
rsync -av --include='*.jpg' --exclude='*' /source/ /dest/

# Partial transfer (kesintide devam)
rsync -avP /source/ /dest/

# Hard links (incremental backup)
rsync -av --link-dest=/backup/prev /source/ /backup/current/

# Checksum-based (modification time yerine)
rsync -avc /source/ /dest/

# Max size limit
rsync -av --max-size=10M /source/ /dest/

# Stats
rsync -av --stats /source/ /dest/

# Log file
rsync -av --log-file=rsync.log /source/ /dest/
```

---

### rsync Backup Script

```bash
#!/bin/bash
# incremental-backup.sh

SOURCE="/home/"
DEST="/backup/"
DATE=$(date +%Y%m%d)
PREV_BACKUP="$DEST/latest"
CURRENT_BACKUP="$DEST/$DATE"

# Hard-link incremental backup
rsync -av --delete \
    --link-dest="$PREV_BACKUP" \
    --exclude="*.tmp" \
    --exclude=".cache" \
    --log-file="/var/log/backup.log" \
    "$SOURCE" "$CURRENT_BACKUP"

# Update 'latest' symlink
rm -f "$PREV_BACKUP"
ln -s "$CURRENT_BACKUP" "$PREV_BACKUP"

# Rotate old backups (30 günden eski sil)
find "$DEST" -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;

echo "Backup completed: $CURRENT_BACKUP"
```

---

## Database Backup

### MySQL/MariaDB

```bash
# Single database
mysqldump -u root -p database_name > backup.sql

# All databases
mysqldump -u root -p --all-databases > all_databases.sql

# Specific tables
mysqldump -u root -p database_name table1 table2 > tables.sql

# Compress
mysqldump -u root -p database_name | gzip > backup.sql.gz

# Remote database
mysqldump -h remote.server -u root -p database_name > backup.sql

# With routines, triggers, events
mysqldump -u root -p --routines --triggers --events database_name > backup.sql

# Lock-free (InnoDB)
mysqldump -u root -p --single-transaction database_name > backup.sql

# Restore
mysql -u root -p database_name < backup.sql
zcat backup.sql.gz | mysql -u root -p database_name
```

**Automated MySQL backup:**
```bash
#!/bin/bash
# mysql-backup.sh

DB_USER="root"
DB_PASS="password"
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Backup all databases
mysqldump -u "$DB_USER" -p"$DB_PASS" --all-databases | gzip > "$BACKUP_DIR/all_databases_$DATE.sql.gz"

# Rotate old backups (7 günden eski sil)
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete

echo "MySQL backup completed: $DATE"
```

---

### PostgreSQL

```bash
# Single database
pg_dump -U postgres database_name > backup.sql

# Custom format (better compression)
pg_dump -U postgres -F c database_name > backup.dump

# All databases
pg_dumpall -U postgres > all_databases.sql

# Specific tables
pg_dump -U postgres -t table1 -t table2 database_name > tables.sql

# Compress
pg_dump -U postgres database_name | gzip > backup.sql.gz

# Remote database
pg_dump -h remote.server -U postgres database_name > backup.sql

# Restore
psql -U postgres database_name < backup.sql
pg_restore -U postgres -d database_name backup.dump
```

---

## System Backup

### dd - Disk Image

```bash
# Disk/partition image
sudo dd if=/dev/sda of=/backup/sda.img bs=4M status=progress

# Compress with gzip
sudo dd if=/dev/sda bs=4M | gzip > /backup/sda.img.gz

# Clone disk
sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress

# Restore
sudo dd if=/backup/sda.img of=/dev/sda bs=4M status=progress
zcat /backup/sda.img.gz | sudo dd of=/dev/sda bs=4M
```

---

### partclone

Sadece kullanılan blokları backup al (dd'den hızlı).

```bash
# Kur
sudo apt install partclone

# ext4 partition backup
sudo partclone.ext4 -c -s /dev/sda1 -o /backup/sda1.img

# Compress
sudo partclone.ext4 -c -s /dev/sda1 | gzip > /backup/sda1.img.gz

# Restore
sudo partclone.restore -s /backup/sda1.img -o /dev/sda1
```

---

### Clonezilla

Full system clone (GUI).

```bash
# ISO download
wget https://sourceforge.net/projects/clonezilla/files/clonezilla_live_stable/

# USB'ye yaz
sudo dd if=clonezilla.iso of=/dev/sdb bs=4M status=progress

# USB'den boot et ve wizard takip et
```

---

## Automated Backup

### Cron ile Backup

```bash
# Crontab düzenle
crontab -e

# Günlük 2:00'da backup
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

# Haftalık Pazar 3:00'te
0 3 * * 0 /usr/local/bin/weekly-backup.sh

# Aylık 1. gün 4:00'te
0 4 1 * * /usr/local/bin/monthly-backup.sh
```

---

### systemd Timer ile Backup

**backup.service:**
```bash
sudo nano /etc/systemd/system/backup.service
```

```ini
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=root
```

**backup.timer:**
```bash
sudo nano /etc/systemd/system/backup.timer
```

```ini
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
# Aktif et
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
sudo systemctl list-timers
```

---

## Cloud Backup

### rclone (Cloud Storage Sync)

```bash
# Kur
sudo apt install rclone

# Config
rclone config

# Google Drive sync
rclone sync /local/path remote:backup

# S3 sync
rclone sync /local/path s3:bucket/backup

# Dry run
rclone sync --dry-run /local/path remote:backup

# Bandwidth limit
rclone sync --bwlimit 1M /local/path remote:backup

# Exclude
rclone sync --exclude "*.tmp" /local/path remote:backup

# Encrypted
rclone sync /local/path crypt:backup
```

**rclone cron:**
```bash
# Günlük cloud backup
0 3 * * * /usr/bin/rclone sync /data gdrive:backup >> /var/log/rclone.log 2>&1
```

---

### AWS S3 Backup

```bash
# AWS CLI kur
sudo apt install awscli

# Config
aws configure

# Upload
aws s3 sync /local/path s3://bucket/backup/

# Download
aws s3 sync s3://bucket/backup/ /local/path/

# Compress ve upload
tar -czf - /data | aws s3 cp - s3://bucket/backup/data.tar.gz

# Lifecycle policy (otomatik eski backup silme)
aws s3api put-bucket-lifecycle-configuration --bucket mybackup --lifecycle-configuration file://lifecycle.json
```

**lifecycle.json:**
```json
{
  "Rules": [
    {
      "Id": "Delete old backups",
      "Status": "Enabled",
      "Prefix": "backup/",
      "Expiration": {
        "Days": 30
      }
    }
  ]
}
```

---

## Backup Rotation

### Grandfather-Father-Son (GFS)

```
Daily: 7 gün
Weekly: 4 hafta
Monthly: 12 ay
Yearly: indefinite
```

**GFS backup script:**
```bash
#!/bin/bash

SOURCE="/data"
BACKUP_ROOT="/backup"
DATE=$(date +%Y%m%d)
DAY_OF_WEEK=$(date +%u)  # 1=Monday, 7=Sunday
DAY_OF_MONTH=$(date +%d)

# Daily backup
DAILY_DIR="$BACKUP_ROOT/daily/$DATE"
rsync -av --delete "$SOURCE/" "$DAILY_DIR/"

# Weekly backup (Sunday)
if [ "$DAY_OF_WEEK" -eq 7 ]; then
    WEEKLY_DIR="$BACKUP_ROOT/weekly/$DATE"
    cp -al "$DAILY_DIR" "$WEEKLY_DIR"
fi

# Monthly backup (1st of month)
if [ "$DAY_OF_MONTH" -eq "01" ]; then
    MONTHLY_DIR="$BACKUP_ROOT/monthly/$DATE"
    cp -al "$DAILY_DIR" "$MONTHLY_DIR"
fi

# Rotate
find "$BACKUP_ROOT/daily" -maxdepth 1 -type d -mtime +7 -exec rm -rf {} \;
find "$BACKUP_ROOT/weekly" -maxdepth 1 -type d -mtime +28 -exec rm -rf {} \;
find "$BACKUP_ROOT/monthly" -maxdepth 1 -type d -mtime +365 -exec rm -rf {} \;
```

---

## Backup Verification

### Backup Test

```bash
#!/bin/bash
# verify-backup.sh

BACKUP_FILE="/backup/data.tar.gz"

# Checksum
if [ -f "$BACKUP_FILE.md5" ]; then
    md5sum -c "$BACKUP_FILE.md5"
    if [ $? -eq 0 ]; then
        echo "✅ Checksum OK"
    else
        echo "❌ Checksum FAILED"
        exit 1
    fi
fi

# Test extract
mkdir -p /tmp/backup-test
tar -xzf "$BACKUP_FILE" -C /tmp/backup-test --exclude='*' -O > /dev/null
if [ $? -eq 0 ]; then
    echo "✅ Archive integrity OK"
else
    echo "❌ Archive corrupted"
    exit 1
fi

rm -rf /tmp/backup-test
```

---

### Checksum Oluşturma

```bash
# MD5
md5sum backup.tar.gz > backup.tar.gz.md5

# SHA256 (daha güvenli)
sha256sum backup.tar.gz > backup.tar.gz.sha256

# Verify
md5sum -c backup.tar.gz.md5
sha256sum -c backup.tar.gz.sha256
```

---

## Disaster Recovery

### Full System Restore Plan

**1. Bootable Rescue Media:**
```bash
# System Rescue CD
# Download: https://www.system-rescue.org/
```

**2. Disk Restore:**
```bash
# Boot rescue media
# Partition oluştur
sudo fdisk /dev/sda

# Filesystem oluştur
sudo mkfs.ext4 /dev/sda1

# Mount
sudo mount /dev/sda1 /mnt

# Backup restore
sudo tar -xzvf /backup/system.tar.gz -C /mnt

# GRUB kurulumu
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
grub-install /dev/sda
update-grub
exit

# Reboot
sudo reboot
```

---

### Bare Metal Recovery

```bash
#!/bin/bash
# bare-metal-backup.sh

# System config backup
tar -czf /backup/etc-backup.tar.gz /etc

# Installed packages (Debian/Ubuntu)
dpkg --get-selections > /backup/packages.list

# User home directories
tar -czf /backup/home-backup.tar.gz /home

# Database dump
mysqldump -u root -p --all-databases | gzip > /backup/mysql.sql.gz
```

**Restore:**
```bash
# Restore packages
sudo dpkg --set-selections < /backup/packages.list
sudo apt-get dselect-upgrade

# Restore configs
sudo tar -xzf /backup/etc-backup.tar.gz -C /

# Restore homes
sudo tar -xzf /backup/home-backup.tar.gz -C /

# Restore database
zcat /backup/mysql.sql.gz | mysql -u root -p
```

---

## Backup Monitoring

### Email Notification

```bash
#!/bin/bash
# backup-with-notification.sh

EMAIL="admin@example.com"
LOGFILE="/var/log/backup.log"

# Backup
if /usr/local/bin/backup.sh >> "$LOGFILE" 2>&1; then
    STATUS="SUCCESS"
    SUBJECT="✅ Backup Success"
else
    STATUS="FAILED"
    SUBJECT="❌ Backup Failed"
fi

# Email gönder
echo "Backup status: $STATUS" | mail -s "$SUBJECT" "$EMAIL"
```

---

### Backup Size Monitoring

```bash
#!/bin/bash
# check-backup-size.sh

BACKUP_DIR="/backup"
MIN_SIZE=1000000  # 1MB (bytes)

LATEST_BACKUP=$(ls -t "$BACKUP_DIR"/*.tar.gz | head -1)
SIZE=$(stat -c%s "$LATEST_BACKUP")

if [ "$SIZE" -lt "$MIN_SIZE" ]; then
    echo "❌ Backup suspiciously small: $SIZE bytes"
    echo "Backup file might be corrupted!" | mail -s "Backup Alert" admin@example.com
    exit 1
else
    echo "✅ Backup size OK: $SIZE bytes"
fi
```

---

## Best Practices

### ✅ Yapılması Gerekenler

1. **3-2-1 rule**: 3 kopya, 2 media, 1 offsite
2. **Otomatik backup**: Cron/systemd timer
3. **Compression**: gzip/bzip2/xz
4. **Encryption**: GPG/LUKS (sensitive data)
5. **Verification**: Checksum, test restore
6. **Rotation**: GFS veya custom rotation
7. **Monitoring**: Email alerts, log tracking
8. **Documentation**: Restore procedure dokümante et
9. **Test restore**: Düzenli restore testleri
10. **Offsite backup**: Cloud veya remote location

---

### ❌ Yapılmaması Gerekenler

1. ❌ Backup'ı test etmemek
2. ❌ Sadece lokal backup
3. ❌ Manuel backup'a güvenmek
4. ❌ Eski backupları silmemek
5. ❌ Backup'ı encrypt etmemek (sensitive data)
6. ❌ Checksum oluşturmamak
7. ❌ Rotation yapmamak (disk dolar)

---

## 🎯 Özet

| Tool | Kullanım | Örnek |
|------|----------|-------|
| tar | Archive | `tar -czvf backup.tar.gz /data` |
| rsync | Sync | `rsync -avz /src/ /dest/` |
| mysqldump | MySQL backup | `mysqldump -u root -p db > backup.sql` |
| pg_dump | PostgreSQL backup | `pg_dump db > backup.sql` |
| rclone | Cloud sync | `rclone sync /data remote:backup` |
| dd | Disk image | `dd if=/dev/sda of=disk.img` |

---

## 🎓 Alıştırmalar

1. /home dizinini tar ile yedekle
2. rsync ile incremental backup script yaz
3. MySQL database'i dump al
4. Cloud'a rclone ile backup gönder
5. Cron ile günlük otomatik backup kur
6. GFS rotation script yaz
7. Backup verification script yaz
8. Email notification ekle
9. Full system backup al (bare metal)
10. Restore testini gerçekleştir

---

**Sonraki Bölüm:** [16-Web-Serverleri.md](16-Web-Serverleri.md) → Nginx, Apache, SSL/TLS, reverse proxy, load balancing
