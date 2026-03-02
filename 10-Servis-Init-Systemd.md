# 🐧 LINUX REHBERİ — BÖLÜM 10: SERVİS VE SYSTEMD YÖNETİMİ

Systemd derinlemesine: unit dosyaları, target'lar, timer'lar, systemctl mastery.

---

## Systemd Nedir?

**Systemd**: Modern Linux init sistemi (PID 1).

### Özellikler

- ✅ Paralel servis başlatma (hızlı boot)
- ✅ On-demand activation
- ✅ Dependency management
- ✅ Service health monitoring
- ✅ Unified logging (journald)
- ✅ Gelişmiş resource control (cgroups)

### Systemd Components

```bash
# Ana bileşenler
systemctl      # Servis yönetimi
journalctl     # Log viewing
systemd-analyze  # Performance analizi
hostnamectl    # Hostname yönetimi
timedatectl    # Saat yönetimi
localectl      # Locale yönetimi
loginctl       # Session yönetimi
```

---

## systemctl - Servis Yönetimi

### Temel Komutlar

```bash
# Servis başlat
sudo systemctl start nginx

# Servis durdur
sudo systemctl stop nginx

# Servis yeniden başlat
sudo systemctl restart nginx

# Servis reload (config değişikliği)
sudo systemctl reload nginx

# Reload veya restart (hangisi varsa)
sudo systemctl reload-or-restart nginx

# Servis durumu
systemctl status nginx

# Aktif mi?
systemctl is-active nginx

# Enabled mi?
systemctl is-enabled nginx

# Failed mi?
systemctl is-failed nginx
```

---

### Enable/Disable (Boot)

```bash
# Boot'ta başlasın
sudo systemctl enable nginx

# Boot'ta başlamasın
sudo systemctl disable nginx

# Enable + start
sudo systemctl enable --now nginx

# Disable + stop
sudo systemctl disable --now nginx

# Mask (tamamen engelle)
sudo systemctl mask nginx

# Unmask
sudo systemctl unmask nginx
```

---

### Servis Listeleme

```bash
# Tüm servisler
systemctl list-units --type=service

# Sadece running
systemctl list-units --type=service --state=running

# Sadece failed
systemctl list-units --type=service --state=failed

# Tüm servislerin enable/disable durumu
systemctl list-unit-files --type=service

# Tüm unit tipleri
systemctl list-units

# Sadece aktif
systemctl list-units --state=active

# Sadece failed
systemctl --failed
```

---

### Dependency Görüntüleme

```bash
# Servisin bağımlılıkları
systemctl list-dependencies nginx

# Ters dependency (kime bağlı)
systemctl list-dependencies --reverse nginx

# Tree format
systemctl list-dependencies --plain nginx
```

---

## Unit Dosyaları

### Unit Konumları

```bash
# System unit'leri
/lib/systemd/system/
/usr/lib/systemd/system/

# Custom/override unit'ler
/etc/systemd/system/

# Runtime unit'ler
/run/systemd/system/

# User unit'leri
~/.config/systemd/user/
```

**Priority sırası:**
1. `/etc/systemd/system/` (en yüksek)
2. `/run/systemd/system/`
3. `/usr/lib/systemd/system/`

---

### Service Unit Anatomi

```ini
[Unit]
Description=My Application
Documentation=https://example.com/docs
After=network.target syslog.target
Requires=postgresql.service
Wants=redis.service

[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -TERM $MAINPID
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
Environment="DEBUG=false"
EnvironmentFile=/etc/myapp/config

[Install]
WantedBy=multi-user.target
```

---

### [Unit] Section

```ini
[Unit]
# Açıklama
Description=My Web Server

# Dokümantasyon
Documentation=man:nginx(8)
Documentation=https://nginx.org/docs

# Bağımlılıklar
Requires=postgresql.service    # Zorunlu (bu fail olursa bu da fail)
Wants=redis.service           # Opsiyonel (fail olsa da devam)
BindsTo=network.target        # Sıkı bağlılık

# Sıralama
Before=web.service            # Bu şeyden önce başla
After=network.target          # Bu şeyden sonra başla

# Çakışma
Conflicts=apache2.service     # İkisi birlikte olamaz

# Condition (koşullu başlatma)
ConditionPathExists=/opt/myapp
ConditionFileNotEmpty=/etc/myapp/config
```

---

### [Service] Section

#### Service Type

```ini
[Service]
# Type belirleme
Type=simple      # Default, ExecStart main process
Type=forking     # Parent fork yapıp exit olur (klasik daemon)
Type=oneshot     # Bir kez çalışır ve exit olur
Type=notify      # systemd'ye bildirim gönderir (sd_notify)
Type=dbus        # D-Bus üzerinden aktif olur
Type=idle        # Diğer joblar bitmeden başlamaz
```

**Type Örnekleri:**

```ini
# simple: Direkt çalışır
[Service]
Type=simple
ExecStart=/usr/bin/myapp

# forking: Klasik daemon (nginx, apache)
[Service]
Type=forking
PIDFile=/run/myapp.pid
ExecStart=/usr/bin/myapp --daemon

# oneshot: Script gibi
[Service]
Type=oneshot
ExecStart=/usr/local/bin/setup.sh
RemainAfterExit=yes

# notify: Python uygulama (systemd.daemon kullanarak)
[Service]
Type=notify
ExecStart=/usr/bin/python3 /opt/myapp/app.py
NotifyAccess=main
```

---

#### ExecStart/Stop/Reload

```ini
[Service]
# Ana komut
ExecStart=/usr/bin/myapp --config /etc/myapp.conf

# Başlamadan önce (pre)
ExecStartPre=/usr/bin/check-config.sh

# Başladıktan sonra (post)
ExecStartPost=/usr/bin/notify-started.sh

# Reload (SIGHUP yerine)
ExecReload=/usr/bin/myapp --reload
ExecReload=/bin/kill -HUP $MAINPID

# Stop
ExecStop=/usr/bin/myapp --shutdown
ExecStop=/bin/kill -TERM $MAINPID

# Stop sonrası
ExecStopPost=/usr/bin/cleanup.sh

# Timeout (varsayılan 90s)
TimeoutStartSec=60
TimeoutStopSec=30
```

---

#### Restart Politikası

```ini
[Service]
# Restart davranışı
Restart=no           # Restart yapma (default)
Restart=always       # Her zaman restart
Restart=on-success   # Sadece başarılı exit'te restart
Restart=on-failure   # Sadece fail'de restart
Restart=on-abnormal  # Signal, timeout vb. restart
Restart=on-abort     # Sadece unclean signal'de restart
Restart=on-watchdog  # Watchdog timeout'ta restart

# Restart gecikmesi
RestartSec=5

# Restart limiti (10 saniyede 5 kez deneme)
StartLimitBurst=5
StartLimitIntervalSec=10
```

---

#### User/Group

```ini
[Service]
# Hangi kullanıcıyla çalışsın
User=appuser
Group=appgroup

# veya UID/GID
User=1000
Group=1000

# Supplementary groups
SupplementaryGroups=docker ssl-cert
```

---

#### WorkingDirectory

```ini
[Service]
WorkingDirectory=/opt/myapp
```

---

#### Environment Variables

```ini
[Service]
# Direkt tanımlama
Environment="DEBUG=true"
Environment="PORT=8080"

# Dosyadan yükle
EnvironmentFile=/etc/myapp/config

# Multiple files
EnvironmentFile=/etc/default/myapp
EnvironmentFile=/etc/myapp/secrets
```

**Environment file format (/etc/myapp/config):**
```bash
DEBUG=true
PORT=8080
DATABASE_URL=postgres://localhost/mydb
```

---

#### Standard Output/Error

```ini
[Service]
# Çıktı yönlendirme
StandardOutput=journal     # journald'ye (default)
StandardError=journal

StandardOutput=file:/var/log/myapp.log
StandardError=file:/var/log/myapp-error.log

StandardOutput=null        # Kapat
StandardError=inherit      # Parent'tan devral
```

---

#### Security (Sandboxing)

```ini
[Service]
# Filesystem isolation
ProtectSystem=strict       # / read-only
ProtectHome=true          # /home, /root, /run/user kapat
ReadWritePaths=/var/lib/myapp  # Sadece buna yazabilir

# Private tmp
PrivateTmp=yes

# Network isolation
PrivateDevices=yes        # /dev erişimi kapat
PrivateNetwork=yes        # Network stack kapat

# Capabilities kısıtla
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE

# Syscall filtering
SystemCallFilter=@system-service
SystemCallFilter=~@privileged @resources

# No new privileges
NoNewPrivileges=yes

# Root directory
RootDirectory=/srv/chroot

# Read-only paths
ReadOnlyPaths=/etc /usr

# Inaccessible paths
InaccessiblePaths=/home /root
```

---

### [Install] Section

```ini
[Install]
# Hangi target'a bağlı?
WantedBy=multi-user.target   # systemctl enable'da

# Alias (alternatif isim)
Alias=myapp.service

# Also enable
Also=myapp-worker.service
```

---

### Unit Dosyası Oluşturma

```bash
# Custom service oluştur
sudo nano /etc/systemd/system/myapp.service
```

**Örnek: Python Flask App**
```ini
[Unit]
Description=Flask Web Application
After=network.target

[Service]
Type=simple
User=flask
Group=flask
WorkingDirectory=/opt/flask-app
Environment="FLASK_APP=app.py"
ExecStart=/usr/bin/python3 -m flask run --host=0.0.0.0 --port=5000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
# Reload systemd
sudo systemctl daemon-reload

# Servisi başlat
sudo systemctl start myapp

# Enable et
sudo systemctl enable myapp

# Durumu kontrol et
systemctl status myapp
```

---

### Unit Override

Mevcut unit'i değiştirmeden override et.

```bash
# Override dizini oluştur
sudo systemctl edit nginx

# Veya manuel
sudo mkdir -p /etc/systemd/system/nginx.service.d
sudo nano /etc/systemd/system/nginx.service.d/override.conf
```

**override.conf:**
```ini
[Service]
# Restart policy değiştir
Restart=always
RestartSec=5

# Memory limit ekle
MemoryLimit=512M
```

```bash
# Reload
sudo systemctl daemon-reload

# Kontrol et
systemctl cat nginx
systemctl show nginx
```

---

## Target'lar

Target = Runlevel'ların modern karşılığı.

### Varsayılan Target'lar

```bash
# Default target
systemctl get-default

# Default target değiştir
sudo systemctl set-default multi-user.target

# Geçici target değiştir
sudo systemctl isolate rescue.target
```

**Target listesi:**
```bash
# Tüm target'lar
systemctl list-units --type=target

# Önemli target'lar
poweroff.target      # Sistem kapatma
rescue.target        # Single-user mode (kurtarma)
multi-user.target    # Multi-user, no GUI (runlevel 3)
graphical.target     # GUI (runlevel 5)
reboot.target        # Reboot
```

---

### Custom Target

```bash
# Custom target oluştur
sudo nano /etc/systemd/system/mytarget.target
```

```ini
[Unit]
Description=My Custom Target
Requires=multi-user.target
Conflicts=rescue.target

[Install]
WantedBy=multi-user.target
```

---

## Timer Units (Cron Alternatifi)

Zamanlama için systemd timer'ları.

### Timer Oluşturma

**1. Service dosyası:**
```bash
sudo nano /etc/systemd/system/backup.service
```

```ini
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

**2. Timer dosyası:**
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
AccuracySec=1h

[Install]
WantedBy=timers.target
```

**3. Aktif et:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
```

---

### Timer Sözdizimi

```ini
[Timer]
# Absolute time
OnCalendar=daily                # Her gün gece yarısı
OnCalendar=weekly               # Her Pazar gece yarısı
OnCalendar=monthly              # Her ayın 1'i gece yarısı
OnCalendar=*-*-* 10:00:00       # Her gün 10:00
OnCalendar=Mon-Fri 09:00:00     # Hafta içi 09:00
OnCalendar=*-*-01 00:00:00      # Her ayın 1'i

# Relative time
OnBootSec=15min                 # Boot'tan 15 dk sonra
OnUnitActiveSec=1h              # Son aktivasyondan 1 saat sonra
OnUnitInactiveSec=30min         # Inactive olunca 30 dk sonra
OnActiveSec=5min                # Timer başladıktan 5 dk sonra

# Persistent (kapanma durumunda sonra çalıştır)
Persistent=true

# Accuracy (tolerans)
AccuracySec=1min                # ±1 dakika tolerans
```

---

### Timer İzleme

```bash
# Tüm timer'lar
systemctl list-timers

# Detaylı
systemctl list-timers --all

# Belirli timer durumu
systemctl status backup.timer

# Son aktivasyon
journalctl -u backup.timer
journalctl -u backup.service
```

---

## journalctl - Log Yönetimi

### Temel Kullanım

```bash
# Tüm loglar
journalctl

# Son 100 satır
journalctl -n 100

# Canlı takip (tail -f)
journalctl -f

# Ters sıralama (yeniden eskiye)
journalctl -r
```

---

### Servis Bazlı

```bash
# Belirli servis
journalctl -u nginx

# Multiple services
journalctl -u nginx -u mysql

# Canlı takip
journalctl -u nginx -f

# Son N satır
journalctl -u nginx -n 50
```

---

### Zaman Filtreleme

```bash
# Bugün
journalctl --since today

# Dün
journalctl --since yesterday

# Son 1 saat
journalctl --since "1 hour ago"

# Belirli tarih/saat
journalctl --since "2024-01-15 10:00" --until "2024-01-15 12:00"

# Son boot
journalctl -b

# Önceki boot
journalctl -b -1

# Tüm boot'lar
journalctl --list-boots
```

---

### Priority (Log Level)

```bash
# Sadece hatalar
journalctl -p err

# Warning ve üzeri
journalctl -p warning

# Priority levels
# 0: emerg
# 1: alert
# 2: crit
# 3: err
# 4: warning
# 5: notice
# 6: info
# 7: debug

# Belirli priority
journalctl -p 3
```

---

### Output Format

```bash
# JSON format
journalctl -u nginx -o json

# JSON pretty
journalctl -u nginx -o json-pretty

# Verbose (tüm metadata)
journalctl -u nginx -o verbose

# Cat (sadece mesaj)
journalctl -u nginx -o cat

# Short (default)
journalctl -u nginx -o short
```

---

### Disk Kullanımı

```bash
# Journal disk kullanımı
journalctl --disk-usage

# Vacuum (temizleme)
sudo journalctl --vacuum-time=7d    # 7 günden eski sil
sudo journalctl --vacuum-size=100M  # 100 MB'dan fazla sil

# Rotate
sudo journalctl --rotate

# Verify (corruption check)
sudo journalctl --verify
```

---

### journald Config

```bash
# Config dosyası
sudo nano /etc/systemd/journald.conf
```

```ini
[Journal]
# Storage
Storage=persistent          # /var/log/journal (persistent)
Storage=volatile           # /run/log/journal (RAM)
Storage=auto               # Varsa persistent

# Limits
SystemMaxUse=500M          # Max disk kullanımı
SystemMaxFileSize=100M     # Max dosya boyutu
RuntimeMaxUse=100M         # Max RAM kullanımı

# Retention
MaxRetentionSec=1month     # Max saklama süresi
MaxFileSec=1week           # Max dosya yaşı

# Rate limiting
RateLimitIntervalSec=30s
RateLimitBurst=10000
```

```bash
# Restart journald
sudo systemctl restart systemd-journald
```

---

## systemd-analyze - Performance Analizi

### Boot Zamanı

```bash
# Toplam boot zamanı
systemd-analyze

# Servis bazlı boot zamanları
systemd-analyze blame

# Critical chain (en uzun dependency chain)
systemd-analyze critical-chain

# Grafik (SVG)
systemd-analyze plot > boot.svg
```

---

### Service Dependency Grafiği

```bash
# Dependency tree
systemd-analyze dot | dot -Tsvg > dependencies.svg
```

---

### Service Verify

```bash
# Unit dosyası syntax check
systemd-analyze verify /etc/systemd/system/myapp.service
```

---

## Resource Control (cgroups)

Servislere resource limiti.

```ini
[Service]
# CPU limit
CPUQuota=50%               # Max %50 CPU
CPUWeight=100              # CPU priority (1-10000)

# Memory limit
MemoryLimit=512M
MemoryMax=1G

# I/O limit
IOWeight=100
IOReadBandwidthMax=/dev/sda 10M
IOWriteBandwidthMax=/dev/sda 5M

# Task limit
TasksMax=50

# PID limit
PIDLimit=100
```

**Örnek:**
```bash
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Service]
ExecStart=/usr/bin/myapp
MemoryMax=512M
CPUQuota=50%
TasksMax=100
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

---

### Cgroup Monitoring

```bash
# Servisin resource kullanımı
systemctl status myapp

# Detaylı
systemd-cgtop

# Belirli servis
systemctl show myapp --property=MemoryCurrent
systemctl show myapp --property=CPUUsageNSec
```

---

## Socket Activation

On-demand servis başlatma.

### Socket Unit

```bash
sudo nano /etc/systemd/system/myapp.socket
```

```ini
[Unit]
Description=My App Socket

[Socket]
ListenStream=8080
Accept=no

[Install]
WantedBy=sockets.target
```

**Service unit:**
```ini
[Unit]
Description=My App
Requires=myapp.socket

[Service]
Type=simple
ExecStart=/usr/bin/myapp
StandardInput=socket
```

```bash
# Aktif et
sudo systemctl enable --now myapp.socket

# Kontrol et
systemctl list-sockets
```

---

## Path Unit

Dosya değişikliklerinde servis tetikleme.

```bash
sudo nano /etc/systemd/system/watch-config.path
```

```ini
[Unit]
Description=Watch Config File

[Path]
PathModified=/etc/myapp/config.conf
Unit=reload-config.service

[Install]
WantedBy=multi-user.target
```

**Service:**
```ini
[Unit]
Description=Reload Config

[Service]
Type=oneshot
ExecStart=/usr/bin/myapp --reload
```

---

## Pratik Örnekler

### 1. Flask/Django Deployment

```ini
[Unit]
Description=Django Application
After=network.target postgresql.service

[Service]
Type=notify
User=django
Group=www-data
WorkingDirectory=/opt/django-app
Environment="DJANGO_SETTINGS_MODULE=myapp.settings"
ExecStart=/opt/django-app/venv/bin/gunicorn \
    --bind 0.0.0.0:8000 \
    --workers 4 \
    --timeout 60 \
    --access-logfile - \
    --error-logfile - \
    myapp.wsgi:application
Restart=always
RestartSec=5
MemoryMax=1G
CPUQuota=200%

[Install]
WantedBy=multi-user.target
```

---

### 2. Node.js App

```ini
[Unit]
Description=Node.js Application
After=network.target

[Service]
Type=simple
User=nodejs
WorkingDirectory=/opt/node-app
Environment="NODE_ENV=production"
ExecStart=/usr/bin/node /opt/node-app/server.js
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

### 3. Automatic Backup Timer

**backup.sh:**
```bash
#!/bin/bash
tar -czf /backup/backup-$(date +%Y%m%d).tar.gz /data
find /backup -mtime +30 -delete
```

**backup.service:**
```ini
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=root
```

**backup.timer:**
```ini
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

---

## 🎯 Özet

| Komut | Açıklama | Örnek |
|-------|----------|-------|
| `systemctl start` | Servis başlat | `systemctl start nginx` |
| `systemctl enable` | Boot'ta başlat | `systemctl enable nginx` |
| `systemctl status` | Durum | `systemctl status nginx` |
| `systemctl list-units` | Unit listesi | `systemctl list-units --type=service` |
| `journalctl -u` | Servis logları | `journalctl -u nginx -f` |
| `systemctl edit` | Override yap | `systemctl edit nginx` |
| `systemd-analyze` | Performance | `systemd-analyze blame` |

---

## 🎓 Alıştırmalar

1. Custom service oluştur ve başlat
2. Service'e memory limit ekle
3. Timer ile günlük yedekleme zamanla
4. Boot zamanını analiz et
5. journalctl ile belirli tarih aralığında log ara
6. Service'in dependency tree'sini görüntüle
7. Override ile restart policy değiştir
8. Failed servisleri listele ve düzelt
9. Socket activation ile servis oluştur
10. Path unit ile config değişikliğinde reload

---

**Sonraki Bölüm:** [11-Guvenlik.md](11-Guvenlik.md) → Firewall, SELinux, AppArmor, SSH hardening
