# 🐧 LINUX REHBERİ — BÖLÜM 5: KULLANICI VE GRUP YÖNETİMİ

Linux'ta kullanıcı, grup ve yetki yönetimi.

---

## Linux Kullanıcı Sistemi

### Kullanıcı Tipleri

1. **Root (Superuser)**
   - UID: 0
   - En yetkili kullanıcı
   - Her şeyi yapabilir

2. **Sistem Kullanıcıları**
   - UID: 1-999
   - Servisler için (nginx, mysql, www-data)
   - Login yapamaz

3. **Normal Kullanıcılar**
   - UID: 1000+
   - Gerçek insanlar
   - Login yapabilir

---

## Kullanıcı Bilgisi Dosyaları

### /etc/passwd

Kullanıcı bilgileri (şifreler yok artık).

```bash
cat /etc/passwd
# kullanici:x:1000:1000:Ad Soyad:/home/kullanici:/bin/bash
# │           │ │    │    │         │                 └─ Shell
# │           │ │    │    │         └─ Home dizini
# │           │ │    │    └─ Tam isim (GECOS)
# │           │ │    └─ Grup ID (GID)
# │           │ └─ Kullanıcı ID (UID)
# │           └─ Şifre placeholder (x = /etc/shadow'da)
# └─ Kullanıcı adı
```

**Örnekler:**
```
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
yasin:x:1000:1000:Yasin Doe:/home/yasin:/bin/bash
```

---

### /etc/shadow

Şifreler (hash'lenmiş) ve şifre politikaları.

```bash
sudo cat /etc/shadow
# yasin:$6$salt$hash...:19000:0:99999:7:::
# │     │                │    │ │     │
# │     │                │    │ │     └─ Uyarı (gün önce)
# │     │                │    │ └─ Max şifre yaşı
# │     │                │    └─ Min şifre yaşı
# │     │                └─ Son şifre değişikliği (epoch'tan günler)
# │     └─ Şifre hash'i
# └─ Kullanıcı adı
```

**Şifre hash algoritmaları:**
- `$1$` = MD5 (eski, güvensiz)
- `$5$` = SHA-256
- `$6$` = SHA-512 (önerilir)
- `$y$` = yescrypt (yeni)

---

### /etc/group

Grup bilgileri.

```bash
cat /etc/group
# developers:x:1001:yasin,ali,veli
# │          │ │    └─ Grup üyeleri
# │          │ └─ Grup ID (GID)
# │          └─ Şifre placeholder
# └─ Grup adı
```

---

### /etc/gshadow

Grup şifreleri (nadiren kullanılır).

```bash
sudo cat /etc/gshadow
# developers:!::yasin,ali
```

---

## Kullanıcı Yönetimi

### useradd - Kullanıcı Ekle

```bash
# Basit kullanıcı oluştur
sudo useradd yasin

# Tüm detaylarla
sudo useradd -m -s /bin/bash -c "Yasin Doe" -G sudo,developers yasin
# -m: Home dizini oluştur
# -s: Shell belirt
# -c: Tam isim (comment)
# -G: Ek gruplara ekle

# UID belirt
sudo useradd -u 1500 ali

# Özel home dizini
sudo useradd -m -d /opt/appuser appuser

# Sistem kullanıcısı (servis için)
sudo useradd -r -s /usr/sbin/nologin nginx

# Şifresiz (şifre sonra verilecek)
sudo useradd -m -s /bin/bash mehmet
```

**Default ayarlar:**
```bash
# /etc/default/useradd
cat /etc/default/useradd

# Varsayılan değerleri değiştir
sudo useradd -D -s /bin/zsh  # Default shell
```

---

### adduser - İnteraktif Kullanıcı Ekle (Debian/Ubuntu)

Daha kullanıcı dostu wrapper.

```bash
# İnteraktif kullanıcı oluştur
sudo adduser yasin
# Şifre sorar
#Full name sorar
# Home oluşturur
# Grup oluşturur
```

---

### usermod - Kullanıcı Değiştir

```bash
# Gruba ekle
sudo usermod -aG sudo yasin
sudo usermod -aG docker yasin

# Shell değiştir
sudo usermod -s /bin/zsh yasin

# Home dizini değiştir
sudo usermod -d /new/home -m yasin
# -m: Eski dosyaları taşı

# Kullanıcı adı değiştir
sudo usermod -l yeniisim eskiisim

# UID değiştir
sudo usermod -u 2000 yasin

# Kilitle (login engellensin)
sudo usermod -L yasin

# Kilidi aç
sudo usermod -U yasin

# Expire date belirle
sudo usermod -e 2025-12-31 yasin
```

---

### userdel - Kullanıcı Sil

```bash
# Kullanıcı sil
sudo userdel yasin

# Kullanıcı ve home dizinini sil
sudo userdel -r yasin

# Zorla sil (aktif olsa bile)
sudo userdel -f yasin
```

---

### passwd - Şifre Yönetimi

```bash
# Kendi şifreni değiştir
passwd

# Başka kullanıcının şifresini değiştir (root)
sudo passwd yasin

# Şifre kilitle
sudo passwd -l yasin

# Şifre kilidini aç
sudo passwd -u yasin

# Şifreyi expire et (sonraki loginde değiştirsin)
sudo passwd -e yasin

# Şifre politikası göster
sudo passwd -S yasin
# yasin PS 2024-01-15 0 99999 7 -1
```

---

### chage - Şifre Yaşlandırma

```bash
# Şifre politikasını göster
sudo chage -l yasin

# Son kullanma tarihi belirle
sudo chage -E 2025-12-31 yasin

# Min/Max şifre yaşı
sudo chage -m 7 -M 90 yasin
# Min: 7 gün sonra değiştirebilir
# Max: 90 günde bir değiştirmeli

# Uyarı gün sayısı
sudo chage -W 7 yasin

# Şifre hemen expire
sudo chage -d 0 yasin
```

---

## Grup Yönetimi

### groupadd - Grup Oluştur

```bash
# Basit grup
sudo groupadd developers

# GID belirt
sudo groupadd -g 3000 developers

# Sistem grubu
sudo groupadd -r nginx
```

---

### groupmod - Grup Değiştir

```bash
# Grup adı değiştir
sudo groupmod -n yeniisim eskiisim

# GID değiştir
sudo groupmod -g 3500 developers
```

---

### groupdel - Grup Sil

```bash
# Grup sil
sudo groupdel developers
```

---

### gpasswd - Grup Üyeliği

```bash
# Kullanıcıyı gruba ekle
sudo gpasswd -a yasin developers

# Kullanıcıyı gruptan çıkar
sudo gpasswd -d yasin developers

# Grup adminleri belirle
sudo gpasswd -A yasin,ali developers
```

---

## Kullanıcı Bilgilerini Görüntüleme

### id

Kullanıcı ve grup bilgileri.

```bash
# Kendi bilgilerim
id
# uid=1000(yasin) gid=1000(yasin) groups=1000(yasin),27(sudo),999(docker)

# Başka kullanıcı
id ali

# Sadece UID
id -u

# Sadece GID
id -g

# Tüm gruplar
id -G
id -Gn  # İsimlerle
```

---

### whoami

Şu anki kullanıcı.

```bash
whoami
# yasin
```

---

### who

Sisteme login olanlar.

```bash
who
# yasin   pts/0  2024-01-15 10:30 (192.168.1.100)

# Detaylı
who -a

# Sadece kullanıcı sayısı
who -q
```

---

### w

Daha detaylı login bilgisi.

```bash
w
# USER  TTY   FROM       LOGIN@  IDLE  JCPU  PCPU WHAT
# yasin pts/0 192.168... 10:30   0.00  0.05  0.01 vim file.txt
```

---

### last

Geçmiş login kayıtları.

```bash
# Son loginler
last

# Son 10
last -n 10

# Belirli kullanıcı
last yasin

# Başarısız login denemeleri
lastb

# Son reboot
last reboot
```

---

### users

Aktif kullanıcılar.

```bash
users
# yasin ali veli
```

---

## Sudo Yönetimi

### sudo Nedir?

**SuperUser DO** - Geçici root yetkisi.

```bash
# Root yetkisiyle komut çalıştır
sudo apt update

# Root ol
sudo -i
sudo su -

# Başka kullanıcı olarak çalıştır
sudo -u www-data ls /var/www

# Ortam değişkenlerini koru
sudo -E komut
```

---

### /etc/sudoers

Sudo politikaları.

**⚠️ Direkt düzenleme, vi çökmesi gibi durumda kirilebilir. ASLA `nano /etc/sudoers` YAPMA!**

```bash
# Güvenli düzenleme
sudo visudo
```

**Syntax:**
```
kullanici  HOST=(user:group)  COMMANDS
```

**Örnekler:**
```bash
# Root yetkisi (herkes)
yasin ALL=(ALL:ALL) ALL

# Şifresiz sudo
yasin ALL=(ALL) NOPASSWD: ALL

# Sadece belirli komutlar
ali ALL=(ALL) /usr/bin/systemctl, /usr/bin/apt

# Grup yetkisi
%sudo ALL=(ALL:ALL) ALL
%developers ALL=(ALL) /usr/bin/docker

# Host bazlı
yasin webserver=(ALL) ALL
```

**Sudoers.d kullanımı (önerilir):**
```bash
# Ayrı dosya oluştur
sudo visudo -f /etc/sudoers.d/developers

# İçerik
%developers ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```

---

### sudo Logları

```bash
# Sudo loglarını gör
sudo cat /var/log/auth.log | grep sudo

# Son sudo komutları
sudo journalctl _COMM=sudo
```

---

## Kullanıcı Geçişi

### su (Switch User)

```bash
# Root ol
su -
# veya
su - root

# Başka kullanıcı ol
su - yasin

# Sadece shell değiştir (ortamı korur)
su yasin

# Komut çalıştır ve çık
su - yasin -c "ls -la"
```

**su vs sudo:**
- `su`: Root şifresi gerekir
- `sudo`: Kendi şifren yeter

---

## PAM (Pluggable Authentication Modules)

Kimlik doğrulama sistemi.

```bash
# PAM ayarları
ls /etc/pam.d/

# Örnek: SSH için
cat /etc/pam.d/sshd

# Örnek: sudo için
cat /etc/pam.d/sudo
```

---

## Pratik Senaryolar

### Web Developer Kullanıcısı

```bash
# Kullanıcı oluştur
sudo adduser webdev

# Gruplara ekle
sudo usermod -aG www-data,developers webdev

# Sudo yetkisi (sadece web servisleri)
echo "webdev ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx, /usr/bin/systemctl reload nginx" | \
  sudo tee /etc/sudoers.d/webdev

# Web dizinine yetki
sudo chown -R webdev:www-data /var/www/html
sudo chmod -R 775 /var/www/html
```

---

### Database Admin Kullanıcısı

```bash
# Kullanıcı oluştur
sudo adduser dbadmin

# MySQL grubuna ekle
sudo usermod -aG mysql dbadmin

# Sudo yetkisi
echo "dbadmin ALL=(ALL) /usr/bin/systemctl * mysql, /usr/bin/mysql" | \
  sudo tee /etc/sudoers.d/dbadmin
```

---

### Deploy Bot (CI/CD)

```bash
# Servis kullanıcısı (login yok)
sudo useradd -r -m -d /var/lib/deploy -s /bin/bash deploy

# SSH key için
sudo mkdir /var/lib/deploy/.ssh
sudo chmod 700 /var/lib/deploy/.ssh

# Sudo yetkisi (şifresiz)
echo "deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart app, /usr/bin/docker compose" | \
  sudo tee /etc/sudoers.d/deploy

# Sahiplik
sudo chown -R deploy:deploy /var/lib/deploy
```

---

## Kullanıcı Şablonları

### /etc/skel

Yeni kullanıcılar için şablon dizin.

```bash
# İçeriği gör
ls -la /etc/skel

# Yeni kullanıcıya verilecek dosyalar
sudo cp custom_bashrc /etc/skel/.bashrc
sudo cp welcome.txt /etc/skel/

# Artık yeni kullanıcılara bu dosyalar kopyalanır
```

---

## Login Mesajları

### MOTD (Message of the Day)

```bash
# Login sonrası mesaj
sudo nano /etc/motd

# Dinamik MOTD
ls /etc/update-motd.d/

# Yeni script ekle
sudo nano /etc/update-motd.d/99-custom
```

---

### /etc/issue

Login öncesi mesaj (terminal).

```bash
sudo nano /etc/issue
```

---

## Kullanıcı Kısıtlamaları

### Resource Limits

```bash
# /etc/security/limits.conf
sudo nano /etc/security/limits.conf

# Örnekler:
yasin hard nproc 100        # Max 100 process
yasin soft nofile 1024      # Max 1024 açık dosya
@developers hard cpu 60     # Max 60 dakika CPU
```

---

### Disk Quota

```bash
# Quota kur
sudo apt install quota

# Dosya sisteminde quota aktif et
# /etc/fstab'a usrquota,grpquota ekle

# Quota başlat
sudo quotacheck -cum /home
sudo quotaon /home

# Kullanıcıya quota ver
sudo edquota yasin
# Soft: 5GB, Hard: 10GB

# Quota durumu
quota
sudo repquota /home
```

---

## Best Practices

### ✅ Yapılması Gerekenler

1. **Her bir servis için ayrı kullanıcı**
   ```bash
   sudo useradd -r -s /usr/sbin/nologin nginx
   ```

2. **Güçlü şifre politikası**
   ```bash
   sudo chage -M 90 -m 7 -W 7 yasin
   ```

3. **Sudo log tutma**
   ```bash
   # /etc/sudoers'a ekle
   Defaults logfile="/var/log/sudo.log"
   ```

4. **Gereksiz kullanıcıları devre dışı bırak**
   ```bash
   sudo usermod -L old_user
   sudo usermod -e 1 old_user
   ```

5. **Root login'i kapat (SSH)**
   ```bash
   # /etc/ssh/sshd_config
   PermitRootLogin no
   ```

---

### ❌ Yapılmaması Gerekenler

1. ❌ **Herkesin root şifresini bilmesi**

2. ❌ **Şifresiz sudo (production'da)**
   ```bash
   # YAPMA
   yasin ALL=(ALL) NOPASSWD: ALL
   ```

3. ❌ **Servisler normal kullanıcıyla**
   ```bash
   # YAPMA: nginx normal kullanıcıyla çalışmasın
   ```

4. ❌ **Aynı UID birden fazla kullanıcıda**

---

## 🎯 Özet

| Komut | Açıklama | Örnek |
|-------|----------|-------|
| `useradd` | Kullanıcı ekle | `useradd -m yasin` |
| `usermod` | Kullanıcı değiştir | `usermod -aG sudo yasin` |
| `userdel` | Kullanıcı sil | `userdel -r yasin` |
| `passwd` | Şifre değiştir | `passwd yasin` |
| `groupadd` | Grup ekle | `groupadd developers` |
| `gpasswd` | Gruba ekle/çıkar | `gpasswd -a yasin sudo` |
| `id` | Kullanıcı bilgisi | `id yasin` |
| `sudo` | Root yetkisi | `sudo apt update` |
| `visudo` | Sudoers düzenle | `visudo` |

---

## 🎓 Alıştırmalar

1. Yeni bir kullanıcı oluştur, home dizini ve şifre ver
2. Kullanıcıyı `sudo` grubuna ekle
3. Kullanıcının UID ve GID'sini kontrol et
4. Yeni bir grup oluştur ve kullanıcıyı ekle
5. Sudo yetkisi ver (sadece `systemctl` için)
6. Kullanıcının şifre politikasını ayarla (90 gün max)
7. Sistem kullanıcısı oluştur (login yok)
8. `/etc/skel`'e özel dosya ekle ve yeni kullanıcı oluştur
9. Kullanıcıya disk quota ver
10. Login geçmişini görüntüle

---

**Sonraki Bölüm:** [06-Surec-Yonetimi.md](06-Surec-Yonetimi.md) → Process, servis ve zamanlama
