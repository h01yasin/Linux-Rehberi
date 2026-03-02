# 🐧 LINUX REHBERİ — BÖLÜM 11: GÜVENLİK

Linux sistem güvenliği: firewall, SELinux, AppArmor, SSH hardening, auditing.

---

## Güvenlik Katmanları

```
┌─────────────────────────────────────┐
│   Network Security (Firewall)       │
├─────────────────────────────────────┤
│   Access Control (SELinux/AppArmor) │
├─────────────────────────────────────┤
│   User/Process Isolation            │
├─────────────────────────────────────┤
│   File Permissions                  │
├─────────────────────────────────────┤
│   Encryption (Disk, Network)        │
├─────────────────────────────────────┤
│   Auditing & Monitoring             │
└─────────────────────────────────────┘
```

---

## Firewall (iptables, nftables, ufw, firewalld)

### UFW (Uncomplicated Firewall) - Ubuntu

```bash
# Status
sudo ufw status verbose

# Enable/Disable
sudo ufw enable
sudo ufw disable

# Default policy
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Port izni
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443

# Port aralığı
sudo ufw allow 6000:6007/tcp

# IP'ye izin
sudo ufw allow from 192.168.1.100

# IP ve port
sudo ufw allow from 192.168.1.100 to any port 22

# Subnet
sudo ufw allow from 192.168.1.0/24 to any port 3306

# Deny rule
sudo ufw deny 23

# Delete rule
sudo ufw delete allow 80
sudo ufw status numbered
sudo ufw delete 2

# Reset
sudo ufw reset

# Logging
sudo ufw logging on
sudo ufw logging medium
sudo ufw logging high

# App profiles
sudo ufw app list
sudo ufw allow 'Nginx Full'
sudo ufw allow 'OpenSSH'

# Rate limiting (brute-force koruması)
sudo ufw limit ssh
```

---

### firewalld (RHEL/Rocky/Fedora)

```bash
# Status
sudo firewall-cmd --state
sudo systemctl status firewalld

# Zones
sudo firewall-cmd --get-zones
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --set-default-zone=public
sudo firewall-cmd --get-active-zones

# Zone'a servis ekle
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent

# Port ekle
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --zone=public --add-port=1000-2000/tcp --permanent

# IP'ye izin
sudo firewall-cmd --zone=public --add-source=192.168.1.100 --permanent

# IP range
sudo firewall-cmd --zone=public --add-source=192.168.1.0/24 --permanent

# Rich rule
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port port="22" protocol="tcp" accept' --permanent

# Masquerading (NAT)
sudo firewall-cmd --zone=public --add-masquerade --permanent

# Port forwarding
sudo firewall-cmd --zone=public --add-forward-port=port=80:proto=tcp:toport=8080 --permanent

# Custom service tanımla
sudo nano /etc/firewalld/services/myapp.xml
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>MyApp</short>
  <description>My Application</description>
  <port protocol="tcp" port="8080"/>
  <port protocol="udp" port="9090"/>
</service>
```

```bash
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --add-service=myapp --permanent
sudo firewall-cmd --reload

# Kural silme
sudo firewall-cmd --zone=public --remove-service=http --permanent
sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent

# Reload
sudo firewall-cmd --reload

# Tüm zone bilgisi
sudo firewall-cmd --zone=public --list-all

# Panic mode (tüm trafiği kes)
sudo firewall-cmd --panic-on
sudo firewall-cmd --panic-off
```

---

### iptables (Low-level)

```bash
# Kuralları listele
sudo iptables -L -v -n
sudo iptables -L INPUT -v -n

# Tüm trafiğe izin ver (RESET)
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -F  # Flush rules

# Default policy: DROP
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Loopback'e izin ver
sudo iptables -A INPUT -i lo -j ACCEPT

# Established/related
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH izni
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Belirli IP'den
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# IP range
sudo iptables -A INPUT -m iprange --src-range 192.168.1.100-192.168.1.200 -j ACCEPT

# Port range
sudo iptables -A INPUT -p tcp --dport 6000:6007 -j ACCEPT

# ICMP (ping)
sudo iptables -A INPUT -p icmp -j ACCEPT

# Kural sil (line number)
sudo iptables -D INPUT 3

# Kural insert (başa ekle)
sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT

# Kuralları kaydet (Ubuntu)
sudo iptables-save > /etc/iptables/rules.v4

# Kuralları yükle
sudo iptables-restore < /etc/iptables/rules.v4

# Persistent (boot'ta yüklensin)
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

---

## SELinux (Security-Enhanced Linux)

RHEL/Rocky/Fedora'da varsayılan.

### SELinux Modes

- **Enforcing**: Policies enforce edilir (default)
- **Permissive**: Policies enforce edilmez, sadece log
- **Disabled**: SELinux kapalı

---

### SELinux Yönetimi

```bash
# Durum
sestatus

# Geçerli mode
getenforce

# Geçici mode değiştir
sudo setenforce 0  # Permissive
sudo setenforce 1  # Enforcing

# Kalıcı mode değiştir
sudo nano /etc/selinux/config
```

```
SELINUX=enforcing
# veya
SELINUX=permissive
# veya
SELINUX=disabled
```

```bash
# Reboot gerekir
sudo reboot
```

---

### SELinux Context

```bash
# Dosya context
ls -Z /var/www/html/

# Process context
ps auxZ | grep nginx

# User context
id -Z
```

**Context formatı:**
```
user:role:type:level
system_u:object_r:httpd_sys_content_t:s0
```

---

### Context Değiştirme

```bash
# Geçici context değiştir
sudo chcon -t httpd_sys_content_t /var/www/html/index.html

# Recursive
sudo chcon -R -t httpd_sys_content_t /var/www/html/

# Restore default context
sudo restorecon /var/www/html/index.html
sudo restorecon -R /var/www/html/

# Kalıcı context (semanage)
sudo semanage fcontext -a -t httpd_sys_content_t "/custom/web(/.*)?"
sudo restorecon -R /custom/web
```

---

### SELinux Booleans

```bash
# Boolean listesi
getsebool -a

# Boolean değeri
getsebool httpd_can_network_connect

# Boolean değiştir (geçici)
sudo setsebool httpd_can_network_connect on

# Kalıcı
sudo setsebool -P httpd_can_network_connect on

# Önemli booleans:
sudo setsebool -P httpd_can_network_connect on        # HTTP proxy
sudo setsebool -P httpd_can_network_connect_db on     # DB bağlantı
sudo setsebool -P httpd_read_user_content on          # Home dizin okuma
```

---

### SELinux Troubleshooting

```bash
# AVC denied logları
sudo ausearch -m avc -ts recent

# veya journalctl
sudo journalctl -t setroubleshoot

# Detaylı analiz (sealert)
sudo sealert -a /var/log/audit/audit.log

# Port context
sudo semanage port -l | grep http
sudo semanage port -a -t http_port_t -p tcp 8080
```

---

## AppArmor (Ubuntu)

SELinux alternatifi, path-based.

### AppArmor Yönetimi

```bash
# Durum
sudo aa-status

# Enforcement mode
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx

# Complain mode (log only)
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx

# Disable
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx

# Enable
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx

# Reload
sudo systemctl reload apparmor
```

---

### AppArmor Profile

```bash
# Profile konumu
ls /etc/apparmor.d/

# Örnek profile
sudo nano /etc/apparmor.d/usr.bin.myapp
```

```
#include <tunables/global>

/usr/bin/myapp {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  # Capabilities
  capability net_bind_service,
  capability setuid,

  # File access
  /etc/myapp/** r,
  /var/lib/myapp/** rw,
  /var/log/myapp/** w,

  # Network
  network tcp,
  network udp,

  # Execute
  /usr/bin/python3 ix,
}
```

```bash
# Profile yükle
sudo apparmor_parser -r /etc/apparmor.d/usr.bin.myapp

# Syntax check
sudo apparmor_parser /etc/apparmor.d/usr.bin.myapp
```

---

## SSH Hardening

### SSH Server Güvenlik

```bash
# Config düzenle
sudo nano /etc/ssh/sshd_config
```

**Önerilen ayarlar:**
```
# Port değiştir (brute-force azaltır)
Port 2222

# Protocol 2 (eski protocol 1 devre dışı)
Protocol 2

# Root login kapat
PermitRootLogin no

# Şifre kullanma (sadece key)
PasswordAuthentication no
PubkeyAuthentication yes

# Empty password'e izin verme
PermitEmptyPasswords no

# X11 forwarding kapat (gerekmedikçe)
X11Forwarding no

# Login grace time (timeout)
LoginGraceTime 30

# Max auth tries
MaxAuthTries 3

# Max sessions
MaxSessions 2

# Allowed users
AllowUsers yasin ali

# Allowed groups
AllowGroups sshusers

# Banner
Banner /etc/ssh/banner

# Idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable host-based auth
HostbasedAuthentication no

# Disable rhosts
IgnoreRhosts yes

# Strong ciphers
Ciphers aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512,hmac-sha2-256
KexAlgorithms diffie-hellman-group-exchange-sha256

# Log level
LogLevel VERBOSE
```

```bash
# Config test
sudo sshd -t

# Restart SSH
sudo systemctl restart sshd
```

---

### SSH Key Authentication

```bash
# Key oluştur
ssh-keygen -t ed25519 -C "yasin@example.com"
# veya RSA (4096 bit)
ssh-keygen -t rsa -b 4096 -C "yasin@example.com"

# Public key'i sunucuya kopyala
ssh-copy-id -i ~/.ssh/id_ed25519.pub yasin@server

# Manuel kopyalama
cat ~/.ssh/id_ed25519.pub | ssh yasin@server 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'

# Permissions (önemli!)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

---

### SSH Yetkili Keys Yönetimi

```bash
# authorized_keys
nano ~/.ssh/authorized_keys

# Key options
# Command restriction
command="/usr/local/bin/backup.sh" ssh-ed25519 AAAA...

# IP restriction
from="192.168.1.100" ssh-ed25519 AAAA...

# No port forwarding
no-port-forwarding,no-X11-forwarding ssh-ed25519 AAAA...
```

---

### Fail2Ban

SSH brute-force koruması.

```bash
# Kur
sudo apt install fail2ban

# Config
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5
destemail = admin@example.com
action = %(action_mwl)s

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 24h
```

```bash
# Restart
sudo systemctl restart fail2ban

# Status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Banned IP'leri gör
sudo fail2ban-client get sshd banned

# IP'yi unban et
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

---

## Şifreleme

### LUKS (Disk Encryption)

```bash
# Partition şifrele
sudo cryptsetup luksFormat /dev/sdb1

# Aç
sudo cryptsetup luksOpen /dev/sdb1 encrypted_disk

# Format
sudo mkfs.ext4 /dev/mapper/encrypted_disk

# Mount
sudo mount /dev/mapper/encrypted_disk /mnt/encrypted

# Kapat
sudo umount /mnt/encrypted
sudo cryptsetup luksClose encrypted_disk

# /etc/crypttab (otomatik açılsın)
encrypted_disk /dev/sdb1 none luks

# /etc/fstab
/dev/mapper/encrypted_disk /mnt/encrypted ext4 defaults 0 2
```

---

### GPG (File Encryption)

```bash
# Key oluştur
gpg --gen-key

# Key listele
gpg --list-keys
gpg --list-secret-keys

# Dosya şifrele
gpg -e -r "Yasin" file.txt

# Dosya şifre çöz
gpg -d file.txt.gpg > file.txt

# Sign (imzala)
gpg --sign file.txt

# Verify
gpg --verify file.txt.gpg

# Export public key
gpg --export -a "Yasin" > public.key

# Import public key
gpg --import public.key
```

---

## Auditing (auditd)

Sistem olaylarını izleme.

```bash
# Kur
sudo apt install auditd audispd-plugins

# Status
sudo systemctl status auditd

# Kuralları listele
sudo auditctl -l

# Dosya izle
sudo auditctl -w /etc/passwd -p wa -k passwd_changes

# Dizin izle
sudo auditctl -w /etc/ssh/ -p wa -k ssh_config_changes

# Syscall izle
sudo auditctl -a always,exit -F arch=b64 -S open -S openat -k file_open

# User izle
sudo auditctl -a always,exit -F auid=1000 -S all

# Kuralları kaydet
sudo auditctl -w /etc/audit/rules.d/custom.rules

# Logları ara
sudo ausearch -k passwd_changes
sudo ausearch -m USER_LOGIN -ts recent
sudo ausearch -ua 1000

# Report
sudo aureport
sudo aureport --summary
sudo aureport --login
sudo aureport --failed
```

---

## Security Best Practices

### 1. En Az Yetki Prensibi

```bash
# Servisler özel kullanıcıyla
sudo useradd -r -s /usr/sbin/nologin nginx

# Sudo yetkisi sınırlı
yasin ALL=(ALL) /usr/bin/systemctl restart nginx
```

---

### 2. Düzenli Güncellemeler

```bash
# Unattended upgrades (Ubuntu)
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Manuel güncellemeler
sudo apt update && sudo apt upgrade -y
```

---

### 3. SUID/SGID Bitleri İzleme

```bash
# SUID dosyalarını bul
find / -perm -4000 -type f -ls 2>/dev/null

# SGID dosyalarını bul
find / -perm -2000 -type f -ls 2>/dev/null

# Düzenli kontrol (cron)
find / -perm -4000 -o -perm -2000 > /var/log/suid-files.log
```

---

### 4. File Integrity Monitoring (AIDE)

```bash
# Kur
sudo apt install aide

# Database oluştur
sudo aideinit

# Kontrol et
sudo aide --check

# Update database
sudo aide --update
```

---

### 5. Password Politikası

```bash
# PAM password quality
sudo apt install libpam-pwquality

# Config
sudo nano /etc/security/pwquality.conf
```

```
minlen = 12
minclass = 3
maxrepeat = 2
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
```

---

### 6. Two-Factor Authentication (2FA)

```bash
# Google Authenticator
sudo apt install libpam-google-authenticator

# User setup
google-authenticator

# PAM config
sudo nano /etc/pam.d/sshd
```

```
auth required pam_google_authenticator.so
```

```bash
# SSHD config
sudo nano /etc/ssh/sshd_config
```

```
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

---

### 7. Network Security

```bash
# /etc/sysctl.conf
sudo nano /etc/sysctl.conf
```

```bash
# IP forwarding kapat (router değilse)
net.ipv4.ip_forward = 0

# SYN cookies (SYN flood koruması)
net.ipv4.tcp_syncookies = 1

# ICMP redirects kabul etme
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Source routing kapat
net.ipv4.conf.all.accept_source_route = 0

# ICMP echo ignore (ping yanıtlama)
net.ipv4.icmp_echo_ignore_all = 1

# Log martians (şüpheli paketler)
net.ipv4.conf.all.log_martians = 1
```

```bash
# Uygula
sudo sysctl -p
```

---

## Vulnerability Scanning

### Lynis

```bash
# Kur
sudo apt install lynis

# System audit
sudo lynis audit system

# Report
cat /var/log/lynis-report.dat
```

---

### OpenVAS

```bash
# Kur (Ubuntu)
sudo apt install openvas

# Setup
sudo gvm-setup
sudo gvm-start

# Web interface: https://localhost:9392
```

---

## Security Checklist

### ✅ Sistem Güvenliği

- [ ] En son güvenlik güncellemeleri yüklendi
- [ ] Güçlü şifre politikası aktif
- [ ] Gereksiz servisler kapalı
- [ ] Firewall aktif ve yapılandırıldı
- [ ] SSH hardening yapıldı
- [ ] Root login SSH'dan kapalı
- [ ] Fail2Ban aktif
- [ ] SELinux/AppArmor enforcing mode
- [ ] Audit logging aktif
- [ ] Düzenli yedekleme yapılıyor

---

### ✅ User Güvenliği

- [ ] Gereksiz kullanıcılar devre dışı
- [ ] Tüm kullanıcılar güçlü şifre kullanıyor
- [ ] Sudo yetkisi en aza indirildi
- [ ] Password aging politikası aktif
- [ ] 2FA aktif (kritik sistemlerde)

---

### ✅ Network Güvenliği

- [ ] Sadece gerekli portlar açık
- [ ] Varsayılan portlar değiştirildi
- [ ] Gereksiz network servisleri kapalı
- [ ] TLS/SSL güncel versiyonlar
- [ ] Zayıf cipher'lar devre dışı

---

## 🎯 Özet

| Konu | Tool | Örnek |
|------|------|-------|
| Firewall | ufw/firewalld | `ufw allow 22` |
| SELinux | semanage/setsebool | `setsebool -P httpd_can_network_connect on` |
| AppArmor | aa-status | `aa-enforce /etc/apparmor.d/usr.bin.app` |
| SSH | sshd_config | `PermitRootLogin no` |
| Brute-force | fail2ban | `fail2ban-client status sshd` |
| Encryption | LUKS/GPG | `cryptsetup luksFormat /dev/sdb1` |
| Auditing | auditd | `auditctl -w /etc/passwd` |

---

## 🎓 Alıştırmalar

1. Firewall ile sadece SSH, HTTP, HTTPS'e izin ver
2. SSH'ı farklı porta taşı ve root login'i kapat
3. SSH key authentication kur
4. Fail2Ban kur ve yapılandır
5. SELinux/AppArmor enforcing mode'a geçir
6. Partition'ı LUKS ile şifrele
7. auditd ile /etc/passwd değişikliklerini izle
8. Password politikası güçlendir
9. Lynis ile system audit yap
10. 2FA ile SSH login kur

---

**Sonraki Bölüm:** [12-Logging-Monitoring.md](12-Logging-Monitoring.md) → rsyslog, logrotate, monitoring tools
