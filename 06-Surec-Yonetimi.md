# 🐧 LINUX REHBERİ — BÖLÜM 6: SÜREÇ YÖNETİMİ

Linux'ta process (süreç) yönetimi, monitoring, daemon'lar ve servisler.

---

## Process (Süreç) Nedir?

**Process**: Çalışan bir programın instance'ı.

### Process Özellikleri

- **PID** (Process ID): Benzersiz ID
- **PPID** (Parent PID): Parent process ID
- **UID**: Sahibi kullanıcının ID'si
- **GID**: Sahip grubun ID'si
- **TTY**: Terminal (varsa)
- **State**: Durum (Running, Sleeping, Zombie, etc.)
- **Priority/Nice**: CPU önceliği

---

## Process Durumları

| Durum | Kod | Açıklama |
|-------|-----|----------|
| Running | R | CPU'da çalışıyor |
| Sleeping | S | I/O bekliyor (interruptible) |
| Uninterruptible Sleep | D | I/O bekliyor (interrupt edilemez) |
| Stopped | T | Durdurulmuş (SIGSTOP) |
| Zombie | Z | Bitmiş ama parent temizlememiş |
| Dead | X | Ölmüş |

---

## Process Görüntüleme

### ps (Process Status)

Çalışan processler.

```bash
# Sadece kendi processlerim
ps

# Tüm processler (BSD style)
ps aux
# a: Tüm kullanıcılar
# u: Detaylı kullanıcı bilgisi
# x: Terminal olmayan processler

# Tüm processler (Unix style)
ps -ef
# -e: Everything (tüm processler)
# -f: Full format

# Belirli kullanıcının processleri
ps -u yasin

# Process tree (hiyerarşi)
ps auxf
ps -ejH

# Belirli process'i arama
ps aux | grep nginx

# Custom format
ps -eo pid,ppid,user,%cpu,%mem,cmd
```

**Çıktı örneği:**
```
USER  PID  %CPU %MEM   VSZ   RSS TTY  STAT START TIME COMMAND
yasin 1234  2.5  1.2  50000 12000 pts/0 S+ 10:30 0:05 python app.py
```

- **VSZ**: Virtual memory size
- **RSS**: Resident set size (fiziksel RAM)
- **STAT**: State kodu

---

### pstree

Process ağacı.

```bash
# Process tree
pstree

# PID ile
pstree -p

# Belirli kullanıcı
pstree yasin

# Belirli process'ten başla
pstree 1234
```

---

### top

Gerçek zamanlı process monitoring.

```bash
# İnteraktif monitoring
top

# Tuşlar:
# h: Yardım
# k: Process öldür
# r: Renice (öncelik değiştir)
# f: Sütun seç
# M: Bellek bazlı sırala
# P: CPU bazlı sırala
# u: Kullanıcı filtrele
# q: Çık

# Belirli kullanıcı
top -u yasin

# Batch mode (script içinde)
top -b -n 1

# Belirli süre güncelleme
top -d 5  # 5 saniye
```

**Üst kısım açıklama:**
```
top - 10:30:15 up 5 days, 3:15, 2 users, load average: 0.45, 0.35, 0.28
Tasks: 245 total,   1 running, 244 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  2.1 sy,  0.0 ni, 92.0 id,  0.5 wa,  0.0 hi,  0.2 si,  0.0 st
MiB Mem :  15847.5 total,   2340.1 free,   8245.2 used,   5262.2 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   6845.3 avail Mem
```

- **load average**: 1, 5, 15 dk ortalama yük
- **us**: User space CPU
- **sy**: System (kernel) CPU
- **ni**: Nice (low priority)
- **id**: Idle
- **wa**: I/O wait
- **hi**: Hardware interrupts
- **si**: Software interrupts
- **st**: Steal time (virtualization)

---

### htop

Modern, renkli top alternatifi.

```bash
# Kur
sudo apt install htop    # Debian/Ubuntu
sudo dnf install htop    # RHEL/Rocky

# Çalıştır
htop

# Tuşlar:
# F1: Yardım
# F2: Setup
# F3: Search
# F4: Filter
# F5: Tree view
# F6: Sort by
# F9: Kill
# F10: Quit
```

---

### atop

Gelişmiş monitoring (geçmiş kayıtlar).

```bash
# Kur
sudo apt install atop

# Çalıştır
atop

# Geçmiş kayıtlar
atop -r /var/log/atop/atop_20240115
```

---

## Process Yönetimi

### Process Başlatma

#### Foreground

Normal çalıştırma (terminal kitlenir).

```bash
# Foreground'da çalıştır
sleep 60

# Ctrl+C: İptal et (SIGINT)
```

---

#### Background

Arka planda çalıştır (terminal serbest).

```bash
# Arka planda başlat
sleep 60 &
# [1] 1234

# Çalışan jobs
jobs
# [1]+  Running   sleep 60 &

# Job'u fg'ye getir
fg %1

# Job'u bg'ye gönder (Ctrl+Z'den sonra)
bg %1
```

---

#### nohup

Terminal kapansa bile çalışsın.

```bash
# nohup ile çalıştır
nohup python script.py &

# Çıktı: nohup.out

# Özel çıktı dosyası
nohup python script.py > output.log 2>&1 &

# Disown (session'dan ayır)
python script.py &
disown
```

---

### Jobs Yönetimi

```bash
# Çalışan joblar
jobs

# Job'u fg'ye getir
fg %1
fg %%    # Son job

# Job'u bg'ye gönder
Ctrl+Z   # Durdur
bg %1    # Arka planda devam ettir

# Job'u öldür
kill %1
```

---

### Process Durdurma

#### kill

Signal gönder.

```bash
# Graceful shutdown (SIGTERM)
kill 1234

# Zorla öldür (SIGKILL)
kill -9 1234
kill -KILL 1234

# Diğer signaller
kill -SIGHUP 1234   # Reload config
kill -SIGSTOP 1234  # Durdur
kill -SIGCONT 1234  # Devam ettir

# Tüm signalleri listele
kill -l
```

**Önemli Signaller:**
| Signal | Numara | Açıklama |
|--------|--------|----------|
| SIGHUP | 1 | Hang up (config reload) |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGQUIT | 3 | Quit (Ctrl+\) |
| SIGKILL | 9 | Force kill (yakalanamaz) |
| SIGTERM | 15 | Terminate (default) |
| SIGSTOP | 19 | Stop (yakalanamaz) |
| SIGCONT | 18 | Continue |

---

#### killall

İsme göre öldür.

```bash
# İsme göre öldür
killall firefox

# Zorla
killall -9 python

# Kullanıcıya göre
killall -u yasin
```

---

#### pkill

Pattern'e göre öldür.

```bash
# İsim pattern'i
pkill python

# Regex
pkill -f "python.*app.py"

# Kullanıcıya göre
pkill -u yasin

# Signal belirt
pkill -SIGHUP nginx
```

---

#### pgrep

Process ID bul.

```bash
# İsme göre PID bul
pgrep nginx

# Detaylı
pgrep -a nginx

# Kullanıcıya göre
pgrep -u yasin

# Sayısını göster
pgrep -c python
```

---

## Process Önceliği (Nice)

### Nice Value

**-20** (en yüksek) → **19** (en düşük)

Default: **0**

```bash
# Nice value ile başlat
nice -n 10 python script.py

# En düşük öncelik
nice -n 19 backup.sh &

# Yüksek öncelik (root gerekir)
sudo nice -n -10 important_task
```

---

### renice

Çalışan processin önceliğini değiştir.

```bash
# PID'ye göre
renice -n 5 -p 1234

# Kullanıcıya göre
sudo renice -n 10 -u yasin

# Gruba göre
sudo renice -n 15 -g developers
```

---

## Systemd ve Servis Yönetimi

### Systemd Nedir?

Modern init sistem. PID 1.

### systemctl - Servis Yönetimi

```bash
# Servis başlat
sudo systemctl start nginx

# Servis durdur
sudo systemctl stop nginx

# Servis yeniden başlat
sudo systemctl restart nginx

# Config reload (restart etmeden)
sudo systemctl reload nginx

# Graceful restart (bağlantılar korunur)
sudo systemctl reload-or-restart nginx

# Servis durumu
systemctl status nginx

# Boot'ta başlasın
sudo systemctl enable nginx

# Boot'ta başlamasın
sudo systemctl disable nginx

# Enable ve start birlikte
sudo systemctl enable --now nginx
```

---

### Servis Statusu

```bash
# Detaylı status
systemctl status nginx

# Aktif mi?
systemctl is-active nginx

# Enabled mi?
systemctl is-enabled nginx

# Failed servisler
systemctl --failed

# Tüm servisler
systemctl list-units --type=service

# Sadece running
systemctl list-units --type=service --state=running

# Tüm servislerin durumu
systemctl list-unit-files --type=service
```

---

### systemd Logs

```bash
# Servis logları
sudo journalctl -u nginx

# Canlı takip (tail -f gibi)
sudo journalctl -u nginx -f

# Son 100 satır
sudo journalctl -u nginx -n 100

# Belirli tarih/saat
sudo journalctl -u nginx --since "2024-01-15 10:00"
sudo journalctl -u nginx --since "1 hour ago"

# Bugün
sudo journalctl -u nginx --since today

# Boot logları
sudo journalctl -b

# Priority (hata)
sudo journalctl -p err
```

---

### Systemd Unit Dosyaları

```bash
# Unit dosyası konumu
ls /etc/systemd/system/
ls /lib/systemd/system/

# Örnek unit dosyası oluştur
sudo nano /etc/systemd/system/myapp.service
```

**Örnek Service Unit:**
```ini
[Unit]
Description=My Python Application
After=network.target

[Service]
Type=simple
User=yasin
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**Service Types:**
- `simple`: Ana process (default)
- `forking`: Parent process fork yapar ve exit eder
- `oneshot`: Bir kez çalışır ve exit eder
- `notify`: Process hazır olduğunda systemd'ye bildirir

```bash
# Reload systemd (yeni unit okusun)
sudo systemctl daemon-reload

# Servisi başlat
sudo systemctl start myapp

# Enable et
sudo systemctl enable myapp
```

---

### Timer Units (Cron alternatifi)

```bash
# Timer dosyası
sudo nano /etc/systemd/system/backup.timer
```

**Örnek Timer:**
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

**Service dosyası:**
```ini
[Unit]
Description=Backup Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

```bash
# Timer'ı aktif et
sudo systemctl enable --now backup.timer

# Timer'ları listele
systemctl list-timers

# Timer durumu
systemctl status backup.timer
```

---

## Klasik Init (SysVinit)

Eski sistemlerde (Ubuntu 14.04 öncesi).

```bash
# Servis başlat
sudo service nginx start

# Servis durdur
sudo service nginx stop

# Servis restart
sudo service nginx restart

# Servis durumu
sudo service nginx status

# Runlevel yönetimi
sudo update-rc.d nginx defaults
sudo update-rc.d nginx remove
```

---

## Cron - Zamanlanmış Görevler

### crontab Sözdizimi

```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Haftanın günü (0-7, 0=Pazar)
│ │ │ └───── Ay (1-12)
│ │ └─────── Ayın günü (1-31)
│ └───────── Saat (0-23)
└─────────── Dakika (0-59)
```

---

### crontab Kullanımı

```bash
# Crontab düzenle
crontab -e

# Crontab listele
crontab -l

# Crontab sil
crontab -r

# Başka kullanıcının crontab'ı (root)
sudo crontab -u yasin -e
```

---

### Örnekler

```bash
# Her dakika
* * * * * /path/to/script.sh

# Her saat başı
0 * * * * /path/to/script.sh

# Her gün saat 2:30
30 2 * * * /path/to/backup.sh

# Pazartesi-Cuma, 9:00
0 9 * * 1-5 /path/to/workday.sh

# Her 5 dakikada
*/5 * * * * /path/to/check.sh

# Ayın 1'i, gece yarısı
0 0 1 * * /path/to/monthly.sh

# Sistem reboot'unda
@reboot /path/to/startup.sh

# Günlük (midnight)
@daily /path/to/daily.sh

# Haftalık (Pazar gece yarısı)
@weekly /path/to/weekly.sh

# Aylık (Ayın 1'i gece yarısı)
@monthly /path/to/monthly.sh
```

---

### Cron Log

```bash
# Cron logları
sudo tail -f /var/log/cron
sudo journalctl -u cron
```

---

### anacron

Kapalıyken çalışamayan görevleri telafi eder.

```bash
# /etc/anacrontab
cat /etc/anacrontab
```

---

## at - Tek Seferlik Zamanlama

```bash
# Kur
sudo apt install at

# Servis başlat
sudo systemctl start atd

# Görev zamanla
at 14:30
at> /path/to/script.sh
at> Ctrl+D

# Görev zamanla (farklı formatlar)
at now + 5 minutes
at 2:30 PM tomorrow
at 10:00 AM 01/15/2024

# Zamanlanmış görevler
atq

# Görev sil
atrm 1

# Görev detayı
at -c 1
```

---

## Screen ve tmux - Terminal Multiplexer

### screen

```bash
# Kur
sudo apt install screen

# Yeni session
screen

# İsimlendirilmiş session
screen -S mysession

# Detach
Ctrl+A, D

# Session listesi
screen -ls

# Attach
screen -r mysession

# Zorla attach (başka yerden bağlıysa)
screen -d -r mysession

# Session öldür
screen -X -S mysession quit
```

---

### tmux (önerilir)

```bash
# Kur
sudo apt install tmux

# Yeni session
tmux

# İsimlendirilmiş session
tmux new -s mysession

# Detach
Ctrl+B, D

# Session listesi
tmux ls

# Attach
tmux attach -t mysession

# Window yönetimi
Ctrl+B, C    # Yeni window
Ctrl+B, N    # Sonraki window
Ctrl+B, P    # Önceki window
Ctrl+B, 0-9  # Window seç

# Pane yönetimi
Ctrl+B, %    # Dikey split
Ctrl+B, "    # Yatay split
Ctrl+B, O    # Pane değiştir
Ctrl+B, X    # Pane kapat

# Session öldür
tmux kill-session -t mysession
```

---

## Pratik Senaryolar

### 1. High CPU Kullanımı

```bash
# En çok CPU kullanan 10 process
ps aux --sort=-%cpu | head -10

# Gerçek zamanlı
top
# Sonra 'P' bas (CPU sort)

# Belirli process izle
watch -n 1 "ps -p 1234 -o %cpu,%mem,cmd"
```

---

### 2. Memory Leak Bulma

```bash
# En çok RAM kullanan
ps aux --sort=-%mem | head -10

# Process memory detayı
pmap 1234
cat /proc/1234/status
```

---

### 3. Zombie Process Temizleme

```bash
# Zombie'leri bul
ps aux | awk '$8 ~ /Z/ { print $2 }'

# Parent'ı öldür
ps -o ppid= -p <zombie_pid>
kill <parent_pid>
```

---

### 4. Servis Otomatik Restart

```bash
# Systemd service ile
sudo nano /etc/systemd/system/myapp.service

# [Service] bölümüne ekle:
Restart=always
RestartSec=10
```

---

### 5. Uygulamayı Daemon Yap

```bash
# Screen kullan
screen -dmS myapp python app.py

# Veya tmux
tmux new -d -s myapp 'python app.py'

# Veya nohup
nohup python app.py > app.log 2>&1 &

# Veya systemd service (en iyisi)
```

---

## Process Monitoring Tools

```bash
# I/O kullanımı
sudo iotop

# Network bağlantıları
sudo nethogs

# Disk kullanımı
sudo iotop -o  # Sadece aktif I/O

# CPU core bazlı
mpstat -P ALL 1

# Process açık dosyaları
lsof -p 1234

# Process ağ bağlantıları
lsof -i -a -p 1234
```

---

## 🎯 Özet

| Komut | Açıklama | Örnek |
|-------|----------|-------|
| `ps` | Process listesi | `ps aux` |
| `top` | Gerçek zamanlı monitoring | `top` |
| `htop` | İnteraktif monitoring | `htop` |
| `kill` | Process öldür | `kill -9 1234` |
| `pkill` | İsme göre öldür | `pkill python` |
| `nice` | Öncelikle başlat | `nice -n 10 cmd` |
| `renice` | Öncelik değiştir | `renice -n 5 -p 1234` |
| `systemctl` | Servis yönetimi | `systemctl restart nginx` |
| `journalctl` | Systemd logları | `journalctl -u nginx -f` |
| `crontab` | Zamanlanmış görevler | `crontab -e` |

---

## 🎓 Alıştırmalar

1. En çok CPU ve RAM kullanan 5 process'i bul
2. Python processlerini öldür (pkill)
3. Yeni bir systemd service oluştur
4. Timer unit ile günlük backup zamanla
5. Crontab'ta her 10 dakikada script çalıştır
6. tmux session başlat, detach et, tekrar attach et
7. Process'in açık dosyalarını listele (lsof)
8. Zombie process'leri temizle
9. Nginx servisini reload et (restart etmeden)
10. Boot'ta başlayacak servis oluştur

---

**Sonraki Bölüm:** [07-Ag-Yonetimi.md](07-Ag-Yonetimi.md) → Networking, firewall, DNS
