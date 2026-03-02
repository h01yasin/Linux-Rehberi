# 🐧 LINUX REHBERİ — BÖLÜM 3: DOSYA SİSTEMİ VE İZİNLER

Linux dosya sistemi yapısı ve izin yönetimi.

---

## Linux Dosya Sistemi Hiyerarşisi (FHS)

Linux'ta **her şey bir dosyadır:** Normal dosyalar, dizinler, cihazlar, network soketleri...

```
/                           (Root - En üst dizin)
├── bin/                    Temel kullanıcı komutları (ls, cp, mv)
├── boot/                   Boot loader dosyaları (kernel, initrd)
├── dev/                    Cihaz dosyaları (disk, USB, /dev/null)
├── etc/                    Sistem yapılandırma dosyaları
│   ├── network/            Ağ ayarları
│   ├── nginx/              Nginx config
│   └── ssh/                SSH ayarları
├── home/                   Kullanıcı ev dizinleri
│   ├── yasin/              
│   └── ali/
├── lib/                    Sistem kütüphaneleri
├── media/                  Çıkarılabilir medya (USB, CD)
├── mnt/                    Geçici mount noktaları
├── opt/                    Üçüncü parti yazılımlar
├── proc/                   Sanal dosya sistemi (süreç bilgileri)
├── root/                   Root kullanıcısının ev dizini
├── run/                    Runtime veriler
├── sbin/                   Sistem yönetici komutları (reboot, fsck)
├── srv/                    Servis verileri (web, ftp)
├── sys/                    Çekirdek ve donanım bilgileri
├── tmp/                    Geçici dosyalar (her boot'ta silinir)
├── usr/                    Kullanıcı programları ve veriler
│   ├── bin/                Kullanıcı komutları
│   ├── lib/                Uygulama kütüphaneleri
│   ├── local/              Yerel olarak kurulan yazılımlar
│   └── share/              Paylaşılan veri (man pages, docs)
└── var/                    Değişken veriler
    ├── log/                Log dosyaları
    ├── www/                Web dosyaları
    └── cache/              Cache verileri
```

---

### Önemli Dizinler Detaylı

#### /etc - Yapılandırma Dosyaları

```bash
/etc/passwd                 # Kullanıcı bilgileri
/etc/shadow                 # Şifreler (hash)
/etc/group                  # Grup bilgileri
/etc/hosts                  # Host isimleri
/etc/hostname               # Sistem adı
/etc/fstab                  # Otomatik mount ayarları
/etc/sudoers                # Sudo yetkileri
/etc/ssh/sshd_config        # SSH server ayarları
/etc/nginx/nginx.conf       # Nginx ayarları
```

#### /var - Değişken Veriler

```bash
/var/log/                   # Tüm log dosyaları
/var/log/syslog             # Sistem logları
/var/log/auth.log           # Kimlik doğrulama logları
/var/log/nginx/             # Nginx logları
/var/www/html/              # Web dosyaları
/var/mail/                  # E-postalar
/var/tmp/                   # Geçici dosyalar (kalıcı)
```

#### /proc - Süreç Bilgileri

```bash
/proc/cpuinfo               # CPU bilgisi
/proc/meminfo               # RAM bilgisi
/proc/version               # Kernel versiyonu
/proc/1234/                 # PID 1234 olan sürecin bilgileri
```

#### /dev - Cihaz Dosyaları

```bash
/dev/sda                    # İlk disk
/dev/sda1                   # İlk diskin 1. partitionu
/dev/null                   # Boşluk (çöp kutusu)
/dev/zero                   # Sonsuz sıfırlar
/dev/random                 # Rastgele sayılar
/dev/tty                    # Terminal cihazı
```

---

## Dosya İzinleri

### İzin Türleri

**3 tip izin:**
- `r` (read) = **4** → Okuma
- `w` (write) = **2** → Yazma
- `x` (execute) = **1** → Çalıştırma

**3 grup:**
- **Owner** (Sahip)
- **Group** (Grup)
- **Others** (Diğerleri)

---

### İzinleri Okuma

```bash
ls -l dosya.txt
# -rw-r--r-- 1 yasin developers 1024 Jan 15 10:30 dosya.txt
# │││││││││
# │││││││││
# ││││││└┴┴─ Others (diğerleri): r-- (sadece okuma)
# │││└┴┴──── Group (grup): r-- (sadece okuma)
# │└┴───────  Owner (sahip): rw- (okuma + yazma)
# └───────── Dosya tipi: - (normal dosya)
```

**Dosya tipleri:**
- `-` = Normal dosya
- `d` = Dizin
- `l` = Sembolik link
- `b` = Block device
- `c` = Character device
- `s` = Socket
- `p` = Named pipe

---

### İzin Hesaplama

```
Owner:  r w x  =  4 + 2 + 1  =  7
Group:  r - x  =  4 + 0 + 1  =  5
Others: r - -  =  4 + 0 + 0  =  4

Sonuç: 754
```

**Yaygın izinler:**
- `644` = `-rw-r--r--` → Dosyalar için (owner yazabilir, diğerleri okur)
- `755` = `-rwxr-xr-x` → Scriptler için (owner her şey, diğerleri okur+çalıştırır)
- `777` = `-rwxrwxrwx` → HERKESİN HER ŞEYİ YAPABI (TEHLİKELİ!)
- `700` = `-rwx------` → Sadece owner erişebilir
- `600` = `-rw-------` → Sadece owner okur/yazar

---

### chmod - İzin Değiştirme

#### Sayısal Metod

```bash
# 644 izni ver (rw-r--r--)
chmod 644 dosya.txt

# 755 izni ver (rwxr-xr-x)
chmod 755 script.sh

# 700 izni ver (rwx------)
chmod 700 private.sh

# Recursive (dizin ve içindekiler)
chmod -R 755 /var/www/html
```

#### Sembolik Metod

```bash
# Sahibe execute izni ekle
chmod u+x script.sh
# u=user(owner), +x=execute ekle

# Gruptan write izni kaldır
chmod g-w dosya.txt
# g=group, -w=write kaldır

# Diğerlerine read izni ekle
chmod o+r dosya.txt
# o=others

# Herkese execute ekle
chmod a+x script.sh
# a=all(herkes)

# Sadece sahip okuyabilsin
chmod u=rw,go= dosya.txt
# u=rw (owner: read+write), go= (group ve others: hiçbir şey)

# Birden fazla
chmod u+x,g+w,o-r dosya.txt
```

**Örnekler:**
```bash
# Script çalıştırılabilir yap
chmod +x deploy.sh
chmod 755 deploy.sh

# Private key dosyası
chmod 600 id_rsa

# Public directory
chmod 755 /var/www/html
chmod 644 /var/www/html/index.html
```

---

### chown - Sahiplik Değiştirme

```bash
# Sahibi değiştir
chown yasin dosya.txt

# Hem sahip hem grup
chown yasin:developers dosya.txt

# Sadece grup değiştir
chown :www-data dosya.txt
# veya
chgrp www-data dosya.txt

# Recursive
chown -R www-data:www-data /var/www/html

# Verbose
chown -v yasin dosya.txt
```

**Gerçek örnekler:**
```bash
# Nginx dosyalarının sahibi
sudo chown -R www-data:www-data /var/www/html

# User'ın home dizini
sudo chown -R yasin:yasin /home/yasin

# Docker socket
sudo chown root:docker /var/run/docker.sock
```

---

### chgrp - Grup Değiştirme

```bash
# Grubu değiştir
chgrp developers dosya.txt

# Recursive
chgrp -R www-data /var/www

# Verbose
chgrp -v developers dosya.txt
```

---

## Özel İzinler

### SUID (Set User ID)

Dosya çalıştırıldığında **sahibinin yetkisiyle** çalışır.

```bash
# SUID ekle
chmod u+s dosya
chmod 4755 dosya

# Görünüm
ls -l dosya
# -rwsr-xr-x  (x yerine s)

# Örnek: passwd komutu
ls -l /usr/bin/passwd
# -rwsr-xr-x  root root
# Normal kullanıcı çalıştırdığında root yetkisiyle çalışır
```

---

### SGID (Set Group ID)

Dosya/dizin çalıştırıldığında **grubun yetkisiyle** çalışır.

```bash
# SGID ekle
chmod g+s dizin
chmod 2755 dizin

# Görünüm
ls -ld dizin
# drwxr-sr-x  (x yerine s)

# Dizinde SGID varsa:
# İçinde oluşturulan dosyalar dizinin grubunu alır
```

---

### Sticky Bit

Dizinde sadece **dosya sahibi silebilir**.

```bash
# Sticky bit ekle
chmod +t dizin
chmod 1777 dizin

# Görünüm
ls -ld dizin
# drwxrwxrwt  (x yerine t)

# Örnek: /tmp
ls -ld /tmp
# drwxrwxrwt  (herkes yazabilir ama sadece sahibi silebilir)
```

---

## Umask

**Yeni dosyaların default izinleri**

```bash
# Umask değerini gör
umask
# 0022

# Umask değiştir
umask 0077

# Nasıl hesaplanır?
# Dosya default: 666 (rw-rw-rw-)
# Dizin default:  777 (rwxrwxrwx)
# Umask:          022
# ─────────────────
# Dosya sonuç:    644 (rw-r--r--)
# Dizin sonuç:    755 (rwxr-xr-x)

# Kalıcı yapmak için ~/.bashrc'ye ekle
echo "umask 0077" >> ~/.bashrc
```

---

## ACL (Access Control Lists)

Daha detaylı izin kontrolü.

```bash
# ACL desteği kontrol et
getfacl dosya.txt

# Belirli kullanıcıya izin ver
setfacl -m u:ali:rw dosya.txt

# Belirli gruba izin ver
setfacl -m g:developers:rwx dizin/

# İzin kaldır
setfacl -x u:ali dosya.txt

# Tüm ACL'leri sil
setfacl -b dosya.txt

# Recursive
setfacl -R -m u:ali:rw dizin/

# Default ACL (yeni dosyalar için)
setfacl -d -m u:ali:rw dizin/
```

---

## Sembolik ve Hard Linkler

### Sembolik Link (Soft Link)

Dosyaya kısayol (Windows shortcut gibi).

```bash
# Sembolik link oluştur
ln -s /var/www/html/index.html link.html

# Görünüm
ls -l link.html
# lrwxrwxrwx ... link.html -> /var/www/html/index.html

# Dizin linki
ln -s /var/log logs

# Link güncelleme
ln -sf yeni_hedef link
```

**Özellikler:**
- Orijinal silinirse link bozulur
- Farklı partition'larda olabilir
- Dizinlere link oluşturulabilir

---

### Hard Link

Dosyaya direkt referans (aynı inode).

```bash
# Hard link oluştur
ln dosya.txt hardlink.txt

# İnode numarası aynı
ls -i dosya.txt hardlink.txt
# 123456 dosya.txt
# 123456 hardlink.txt
```

**Özellikler:**
- Orijinal silinse bile link çalışır
- Aynı partition'da olmalı
- Dizinlere yapılamaz

---

## İnode

Her dosyanın benzersiz numarası.

```bash
# İnode numarasını gör
ls -i dosya.txt
# 123456 dosya.txt

# Detaylı inode bilgisi
stat dosya.txt

# İnode'a göre dosya bul
find / -inum 123456
```

---

## Dosya Attributes

Ekstra dosya özellikleri.

```bash
# Attribute listele
lsattr dosya.txt

# Immutable (değiştirilemez, silinemez)
sudo chattr +i dosya.txt
# Artık root bile silemez!

# Immutable kaldır
sudo chattr -i dosya.txt

# Append-only (sadece ekleme)
sudo chattr +a log.txt
# Dosyaya sadece ekleme yapılabilir, silinmez

# Recursive
sudo chattr -R +i dizin/
```

**Attribute'ler:**
- `i` = Immutable (değiştirilemez)
- `a` = Append-only (sadece ekleme)
- `A` = No atime update (erişim zamanı güncellenmesin)
- `c` = Compressed (sıkıştırılmış)
- `d` = No dump (yedeklemeye dahil etme)

---

## Pratik Örnekler

### Web Server İzinleri

```bash
# Nginx/Apache dosyaları
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
```

---

### SSH Key İzinleri

```bash
# Private key
chmod 600 ~/.ssh/id_rsa

# Public key
chmod 644 ~/.ssh/id_rsa.pub

# Authorized keys
chmod 600 ~/.ssh/authorized_keys

# .ssh dizini
chmod 700 ~/.ssh
```

---

### Script İzinleri

```bash
# Script çalıştırılabilir yap
chmod +x script.sh

# Sadece owner çalıştırabilir
chmod 700 script.sh

# Herkes çalıştırabilir
chmod 755 script.sh
```

---

### Shared Directory

```bash
# Ortak dizin oluştur
sudo mkdir /shared
sudo chown root:developers /shared
sudo chmod 2775 /shared  # SGID + rwxrwxr-x

# Artık içinde oluşturulan dosyalar developers grubunda olur
```

---

## Güvenlik Tavsiyeleri

### ✅ Yapılması Gerekenler

1. **Hassas dosyalara 600 izni**
   ```bash
   chmod 600 /home/yasin/.ssh/id_rsa
   chmod 600 /etc/shadow
   ```

2. **Script'ler 755 veya 700**
   ```bash
   chmod 755 /usr/local/bin/backup.sh
   ```

3. **Web dosyaları 644/755**
   ```bash
   find /var/www -type f -exec chmod 644 {} \;
   find /var/www -type d -exec chmod 755 {} \;
   ```

4. **Düzenli izin kontrolü**
   ```bash
   # 777 izinli dosyaları bul
   find / -perm 777 -type f 2>/dev/null
   
   # SUID dosyaları bul
   find / -perm /4000 2>/dev/null
   ```

---

### ❌ Yapılmaması Gerekenler

1. ❌ **777 izni verme** (herkes her şeyi yapabilir!)
   ```bash
   # ASLA YAPMA
   chmod -R 777 /var/www
   ```

2. ❌ **Root'a gereksiz SUID**
   ```bash
   # TEHLİKELİ
   chmod u+s /bin/bash
   ```

3. ❌ **Herkesin /etc'ye yazması**
   ```bash
   # YAPMA
   chmod 777 /etc
   ```

---

## 🎯 Özet

| Komut | Açıklama | Örnek |
|-------|----------|-------|
| `chmod` | İzin değiştir | `chmod 755 dosya` |
| `chown` | Sahip değiştir | `chown user:group dosya` |
| `chgrp` | Grup değiştir | `chgrp www-data dosya` |
| `umask` | Default izinler | `umask 0022` |
| `ln -s` | Sembolik link | `ln -s hedef link` |
| `getfacl` | ACL göster | `getfacl dosya` |
| `setfacl` | ACL ayarla | `setfacl -m u:user:rw dosya` |

**İzin numaraları:**
- `644` → Dosyalar
- `755` → Dizinler ve script'ler
- `600` → Hassas dosyalar
- `700` → Private dizinler

---

## 🎓 Alıştırmalar

1. Bir dosya oluştur ve izinlerini `640` yap
2. Sahipliğini başka bir kullanıcıya ver
3. Sembolik link oluştur
4. Bir dizine SGID özelliği ekle
5. `/tmp` dizininin sticky bit'ini kontrol et
6. 777 izinli dosyaları sistemde ara
7. `chmod u+x,g-w,o-r` komutunun sonucunu hesapla
8. Bir script oluştur, çalıştırılabilir yap ve çalıştır
9. ACL kullanarak bir kullanıcıya özel izin ver
10. Web server dizini için doğru izinleri ayarla

---

**Sonraki Bölüm:** [04-Paket-Yonetimi.md](04-Paket-Yonetimi.md) → Program kurma ve güncelleme
