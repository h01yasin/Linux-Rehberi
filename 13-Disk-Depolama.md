# 🐧 LINUX REHBERİ — BÖLÜM 13: DİSK VE DEPOLAMA YÖNETİMİ

Disk yönetimi, partitioning, LVM, RAID, mount, fstab, disk quotas.

---

## Disk Yapısı

```
┌──────────────────────────────────┐
│   /dev/sda (Physical Disk)       │
├──────────────────────────────────┤
│  /dev/sda1 (Partition 1)         │   → Mount: /boot
│  /dev/sda2 (Partition 2)         │   → Mount: / (root)
│  /dev/sda3 (Partition 3)         │   → Mount: /home
│  /dev/sda4 (Extended)            │
│    ├─ /dev/sda5 (Logical)        │   → Swap
│    └─ /dev/sda6 (Logical)        │   → Mount: /data
└──────────────────────────────────┘
```

---

## Disk Bilgisi

### lsblk - Disk Listesi

```bash
# Basit liste
lsblk

# Detaylı
lsblk -f  # Filesystem tipi
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,FSTYPE

# Tree format
lsblk -T

# Sadece disk (partition yok)
lsblk -d
```

**Çıktı:**
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 465.8G  0 disk 
├─sda1   8:1    0   512M  0 part /boot/efi
├─sda2   8:2    0    50G  0 part /
└─sda3   8:3    0 415.3G  0 part /home
```

---

### fdisk - Disk Bilgisi

```bash
# Disk listesi
sudo fdisk -l

# Belirli disk
sudo fdisk -l /dev/sda

# Partition listesi
sudo fdisk -l /dev/sda | grep ^/dev
```

---

### df - Disk Kullanımı

```bash
# Human-readable
df -h

# İnode kullanımı
df -i

# Filesystem tipi
df -T

# Belirli filesystem
df -h /home

# Exclude tmpfs vb.
df -h -x tmpfs -x devtmpfs
```

---

### du - Dizin Boyutu

```bash
# Dizin boyutu
du -sh /home/yasin

# Alt dizinler
du -h --max-depth=1 /home

# En büyük 10 dizin
du -ah /home | sort -rh | head -10

# Sadece toplam
du -s /home/*

# Exclude pattern
du -sh --exclude=*.log /var
```

---

## Partitioning

### fdisk (MBR - Legacy)

```bash
# Disk aç
sudo fdisk /dev/sdb

# Komutlar:
# m: Yardım
# p: Partition listesi
# n: Yeni partition
# d: Partition sil
# t: Partition tipi değiştir
# w: Kaydet ve çık
# q: Çık (kaydetme)

# Örnek: Yeni partition
sudo fdisk /dev/sdb
# n (new)
# p (primary)
# 1 (partition number)
# [Enter] (first sector default)
# +10G (size)
# w (write)
```

---

### gdisk (GPT - Modern)

GPT (GUID Partition Table) - 2TB'den büyük diskler için.

```bash
# Disk aç
sudo gdisk /dev/sdb

# Komutlar fdisk'e benzer
# ? : Yardım
# p : Partition listesi
# n : Yeni partition
# d : Partition sil
# w : Kaydet
```

---

### parted (Hem MBR hem GPT)

```bash
# İnteraktif
sudo parted /dev/sdb

# Komut satırı
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 0% 100%

# Partition table göster
sudo parted /dev/sdb print

# Partition sil
sudo parted /dev/sdb rm 1
```

---

## Filesystem Oluşturma

### mkfs - Make Filesystem

```bash
# ext4
sudo mkfs.ext4 /dev/sdb1

# ext4 (label ile)
sudo mkfs.ext4 -L "MyData" /dev/sdb1

# xfs
sudo mkfs.xfs /dev/sdb1

# btrfs
sudo mkfs.btrfs /dev/sdb1

# fat32
sudo mkfs.vfat -F 32 /dev/sdb1

# ntfs
sudo mkfs.ntfs /dev/sdb1

# Swap
sudo mkswap /dev/sdb1
sudo swapon /dev/sdb1
```

---

### Filesystem Kontrolü

```bash
# ext4 check
sudo fsck.ext4 /dev/sdb1

# Force check
sudo fsck.ext4 -f /dev/sdb1

# Auto repair
sudo fsck.ext4 -y /dev/sdb1

# xfs check
sudo xfs_repair /dev/sdb1

# Unmount gerekir!
sudo umount /dev/sdb1
sudo fsck /dev/sdb1
```

---

## Mount İşlemleri

### Manuel Mount

```bash
# Mount
sudo mount /dev/sdb1 /mnt/mydisk

# Filesystem tipi belirt
sudo mount -t ext4 /dev/sdb1 /mnt/mydisk

# Read-only
sudo mount -o ro /dev/sdb1 /mnt/mydisk

# Bind mount
sudo mount --bind /source /destination

# Unmount
sudo umount /mnt/mydisk

# Force unmount
sudo umount -f /mnt/mydisk

# Lazy unmount (busy'de kullan)
sudo umount -l /mnt/mydisk
```

---

### /etc/fstab (Kalıcı Mount)

Boot'ta otomatik mount.

```bash
sudo nano /etc/fstab
```

**Format:**
```
<device>  <mount_point>  <type>  <options>  <dump>  <pass>
```

**Örnekler:**
```bash
# UUID ile (önerilir)
UUID=abc123-def456  /mnt/data  ext4  defaults  0  2

# Device ile
/dev/sdb1  /mnt/data  ext4  defaults  0  2

# Label ile
LABEL=MyData  /mnt/data  ext4  defaults  0  2

# Swap
UUID=xyz789  none  swap  sw  0  0

# NFS
192.168.1.100:/share  /mnt/nfs  nfs  defaults  0  0

# CIFS (Windows share)
//192.168.1.100/share  /mnt/windows  cifs  username=user,password=pass  0  0

# tmpfs (RAM disk)
tmpfs  /var/tmp  tmpfs  defaults,size=1G  0  0
```

**Options:**
- `defaults`: rw,suid,dev,exec,auto,nouser,async
- `ro`: Read-only
- `rw`: Read-write
- `noexec`: Execute kısıtla
- `nosuid`: SUID bit devre dışı
- `noatime`: Access time güncelleme (performance)
- `nofail`: Mount hatası boot'u durdurmasın
- `user`: Normal kullanıcı mount edebilir

**dump, pass:**
- `dump`: Yedekleme (0 = hayır, 1 = evet)
- `pass`: fsck sırası (0 = check yok, 1 = root, 2 = diğer)

```bash
# UUID bulma
sudo blkid /dev/sdb1

# fstab test (mount all)
sudo mount -a

# fstab syntax check
sudo findmnt --verify
```

---

## LVM (Logical Volume Manager)

Esnek disk yönetimi.

```
┌────────────────────────────────────┐
│   Physical Volumes (PV)            │
│   /dev/sda1, /dev/sdb1, /dev/sdc1  │
└──────────────┬─────────────────────┘
               │
┌──────────────▼─────────────────────┐
│   Volume Group (VG)                │
│   Total pooled storage             │
└──────────────┬─────────────────────┘
               │
┌──────────────▼─────────────────────┐
│   Logical Volumes (LV)             │
│   /dev/vg0/lv_root                 │
│   /dev/vg0/lv_home                 │
│   /dev/vg0/lv_data                 │
└────────────────────────────────────┘
```

---

### LVM Kurulum

```bash
# Kur
sudo apt install lvm2
```

---

### Physical Volume (PV)

```bash
# PV oluştur
sudo pvcreate /dev/sdb1
sudo pvcreate /dev/sdc1

# PV listesi
sudo pvs
sudo pvdisplay

# PV detayı
sudo pvdisplay /dev/sdb1

# PV sil
sudo pvremove /dev/sdb1
```

---

### Volume Group (VG)

```bash
# VG oluştur
sudo vgcreate vg0 /dev/sdb1 /dev/sdc1

# VG listesi
sudo vgs
sudo vgdisplay

# VG'ye PV ekle
sudo vgextend vg0 /dev/sdd1

# VG'den PV çıkar
sudo vgreduce vg0 /dev/sdd1

# VG adı değiştir
sudo vgrename vg0 vg_data

# VG sil
sudo vgremove vg0
```

---

### Logical Volume (LV)

```bash
# LV oluştur
sudo lvcreate -L 10G -n lv_data vg0
# veya yüzde ile
sudo lvcreate -l 50%FREE -n lv_data vg0

# LV listesi
sudo lvs
sudo lvdisplay

# LV büyüt
sudo lvextend -L +5G /dev/vg0/lv_data
# veya tüm boş alanı kullan
sudo lvextend -l +100%FREE /dev/vg0/lv_data

# Filesystem'i de büyüt
sudo resize2fs /dev/vg0/lv_data  # ext4
sudo xfs_growfs /mnt/data        # xfs

# LV küçült (ext4 only, xfs desteklemez!)
sudo umount /mnt/data
sudo e2fsck -f /dev/vg0/lv_data
sudo resize2fs /dev/vg0/lv_data 5G
sudo lvreduce -L 5G /dev/vg0/lv_data

# LV sil
sudo lvremove /dev/vg0/lv_data
```

---

### LVM Snapshot

```bash
# Snapshot oluştur
sudo lvcreate -L 1G -s -n lv_data_snap /dev/vg0/lv_data

# Snapshot mount
sudo mount /dev/vg0/lv_data_snap /mnt/snapshot

# Snapshot'tan geri dön
sudo lvconvert --merge /dev/vg0/lv_data_snap
# Reboot veya umount gerekir

# Snapshot sil
sudo lvremove /dev/vg0/lv_data_snap
```

---

## RAID

### RAID Seviyeleri

| RAID | Min Disk | Capacity | Redundancy | Performance |
|------|----------|----------|------------|-------------|
| 0    | 2        | 100%     | ❌ Yok     | ⚡ Yüksek   |
| 1    | 2        | 50%      | ✅ Mirror  | 📖 Read hızlı |
| 5    | 3        | 66-94%   | ✅ 1 disk  | ⚖️ Dengeli  |
| 6    | 4        | 50-88%   | ✅ 2 disk  | ⚖️ Dengeli  |
| 10   | 4        | 50%      | ✅ Mirror  | ⚡ En hızlı |

---

### mdadm (Software RAID)

```bash
# Kur
sudo apt install mdadm

# RAID 1 oluştur (mirror)
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# RAID 5 oluştur
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# RAID 10
sudo mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde

# RAID durumu
sudo mdadm --detail /dev/md0
cat /proc/mdstat

# Filesystem oluştur
sudo mkfs.ext4 /dev/md0

# Mount
sudo mount /dev/md0 /mnt/raid

# Config kaydet
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u

# Disk ekle (spare)
sudo mdadm --add /dev/md0 /dev/sdf

# Disk çıkar
sudo mdadm --fail /dev/md0 /dev/sdb
sudo mdadm --remove /dev/md0 /dev/sdb

# RAID durdur
sudo umount /mnt/raid
sudo mdadm --stop /dev/md0

# RAID sil
sudo mdadm --zero-superblock /dev/sdb /dev/sdc
```

---

### RAID Monitoring

```bash
# Email alerts
sudo nano /etc/mdadm/mdadm.conf
```

```
MAILADDR admin@example.com
```

```bash
# Daemon restart
sudo systemctl restart mdadm

# Manuel check
sudo mdadm --detail /dev/md0 | grep -i state
```

---

## Disk Quota

Kullanıcı/grup bazlı disk limitleri.

```bash
# Kur
sudo apt install quota

# fstab'a ekle (usrquota,grpquota)
sudo nano /etc/fstab
```

```
UUID=abc123  /home  ext4  defaults,usrquota,grpquota  0  2
```

```bash
# Remount
sudo mount -o remount /home

# Quota dosyaları oluştur
sudo quotacheck -cum /home
sudo quotacheck -cgm /home

# Quota aktif et
sudo quotaon /home

# Kullanıcıya quota ver
sudo edquota -u yasin
```

**edquota editor:**
```
Disk quotas for user yasin (uid 1000):
  Filesystem  blocks  soft  hard  inodes  soft  hard
  /dev/sda3   10000   50000 60000  1000   5000  6000
```

- **soft**: Uyarı limiti
- **hard**: Kesin limit
- **blocks**: KB cinsinden
- **inodes**: Dosya sayısı

```bash
# Grup quota
sudo edquota -g developers

# Quota template kopyala
sudo edquota -p yasin -u ali

# Quota durumu
quota
quota -u yasin
quota -g developers

# Tüm kullanıcılar
sudo repquota /home
sudo repquota -a  # Tüm filesystemler

# Quota kapat
sudo quotaoff /home
```

---

## Disk I/O Monitoring

### iostat

```bash
# Kur
sudo apt install sysstat

# I/O istatistikleri
iostat

# 2 saniyede bir
iostat 2

# Detaylı
iostat -x 2

# MB cinsinden
iostat -m 2

# Belirli disk
iostat -x /dev/sda 2
```

---

### iotop

```bash
# Kur
sudo apt install iotop

# Real-time I/O
sudo iotop

# Sadece aktif I/O
sudo iotop -o

# Accumulative
sudo iotop -a
```

---

### hdparm

Disk performansı ve ayarları.

```bash
# Disk read speed
sudo hdparm -t /dev/sda

# Cache read speed
sudo hdparm -T /dev/sda

# Disk bilgisi
sudo hdparm -I /dev/sda

# SMART status
sudo smartctl -a /dev/sda
```

---

## Disk Benchmark

### dd

```bash
# Write test
dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=direct

# Read test
dd if=/tmp/testfile of=/dev/null bs=1G count=1 iflag=direct

# Cleanup
rm /tmp/testfile
```

---

### fio

```bash
# Kur
sudo apt install fio

# Random read/write
fio --name=randwrite --ioengine=libaio --iodepth=16 --rw=randwrite --bs=4k --direct=1 --size=1G --numjobs=4 --runtime=60 --group_reporting

# Sequential read
fio --name=seqread --ioengine=libaio --iodepth=16 --rw=read --bs=1M --direct=1 --size=1G --numjobs=1 --runtime=60
```

---

## NFS (Network File System)

### NFS Server

```bash
# Kur
sudo apt install nfs-kernel-server

# Export dizini
sudo mkdir -p /srv/nfs/share
sudo chown nobody:nogroup /srv/nfs/share

# Export config
sudo nano /etc/exports
```

```
/srv/nfs/share  192.168.1.0/24(rw,sync,no_subtree_check)
/home/share     192.168.1.100(rw,sync,no_root_squash)
```

**Options:**
- `rw`: Read-write
- `ro`: Read-only
- `sync`: Synchronous writes
- `async`: Asynchronous writes (faster)
- `no_subtree_check`: Performance
- `no_root_squash`: Root olarak bağlan
- `root_squash`: Root'u nobody yap (default)

```bash
# Export'ları uygula
sudo exportfs -a

# Export listesi
sudo exportfs -v

# Restart
sudo systemctl restart nfs-kernel-server

# Firewall
sudo ufw allow from 192.168.1.0/24 to any port nfs
```

---

### NFS Client

```bash
# Kur
sudo apt install nfs-common

# Manuel mount
sudo mount -t nfs 192.168.1.100:/srv/nfs/share /mnt/nfs

# fstab'a ekle
sudo nano /etc/fstab
```

```
192.168.1.100:/srv/nfs/share  /mnt/nfs  nfs  defaults,_netdev  0  0
```

```bash
# Mount
sudo mount /mnt/nfs

# Durumu gör
df -h | grep nfs
showmount -e 192.168.1.100
```

---

## CIFS/SMB (Windows Share)

```bash
# Kur
sudo apt install cifs-utils

# Manuel mount
sudo mount -t cifs //192.168.1.100/Share /mnt/windows -o username=user,password=pass

# Credentials file (güvenli)
sudo nano /etc/samba-credentials
```

```
username=user
password=pass
domain=WORKGROUP
```

```bash
sudo chmod 600 /etc/samba-credentials

# fstab
sudo nano /etc/fstab
```

```
//192.168.1.100/Share  /mnt/windows  cifs  credentials=/etc/samba-credentials,uid=1000,gid=1000  0  0
```

---

## 🎯 Özet

| Komut | Açıklama | Örnek |
|-------|----------|-------|
| `lsblk` | Disk listesi | `lsblk -f` |
| `df` | Disk kullanımı | `df -h` |
| `du` | Dizin boyutu | `du -sh /home` |
| `fdisk` | Partition | `fdisk /dev/sdb` |
| `mkfs` | Filesystem oluştur | `mkfs.ext4 /dev/sdb1` |
| `mount` | Mount | `mount /dev/sdb1 /mnt` |
| `pvcreate` | LVM PV | `pvcreate /dev/sdb1` |
| `vgcreate` | LVM VG | `vgcreate vg0 /dev/sdb1` |
| `lvcreate` | LVM LV | `lvcreate -L 10G -n lv_data vg0` |
| `mdadm` | Software RAID | `mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc` |

---

## 🎓 Alıştırmalar

1. Yeni disk'e partition oluştur ve ext4 formatla
2. fstab'a kalıcı mount ekle
3. LVM ile 10GB logical volume oluştur
4. LV'yi 15GB'a büyüt
5. LVM snapshot al ve geri dön
6. RAID 1 oluştur ve test et
7. Kullanıcıya 5GB disk quota ver
8. NFS share kur ve client'tan mount et
9. iostat ile disk I/O'yu izle
10. dd ile disk performansını test et

---

**Sonraki Bölüm:** [14-Performance-Tuning.md](14-Performance-Tuning.md) → CPU, memory, I/O tuning, kernel parameters
