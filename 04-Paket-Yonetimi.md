# 🐧 LINUX REHBERİ — BÖLÜM 4: PAKET YÖNETİMİ

Linux'ta program kurma, güncelleme ve kaldırma.

---

## Paket Yöneticisi Nedir?

**Paket yöneticisi**, programları ve bağımlılıklarını otomatik olarak kuran, güncelleyen ve kaldıran araçlardır.

**Windows'tan fark:**
- `.exe` çalıştırmazsın
- Merkezi bir repository'den kurarsın
- Bağımlılıklar otomatik hallolur
- Tek komutla sistem genelinde güncelleme

---

## Dağıtımlara Göre Paket Yöneticileri

| Dağıtım | Paket Formatı | Düşük Seviye | Yüksek Seviye |
|---------|---------------|--------------|----------------|
| **Debian/Ubuntu** | `.deb` | `dpkg` | `apt` / `apt-get` |
| **Red Hat/Rocky/Fedora** | `.rpm` | `rpm` | `dnf` / `yum` |
| **Arch** | `.pkg.tar.zst` | `pacman` | `pacman` |
| **Gentoo** | source | `emerge` | `emerge` |

**Evrensel:**
- `snap` (Canonical)
- `flatpak` (Flathub)
- `AppImage` (portable)

---

## APT (Debian/Ubuntu)

### Temel Komutlar

```bash
# Paket listesini güncelle
sudo apt update

# Tüm paketleri güncelle
sudo apt upgrade

# Yeni versiyonlara geç (çevirdeki yükseltmeler)
sudo apt dist-upgrade
# veya
sudo apt full-upgrade

# Paket kur
sudo apt install nginx

# Birden fazla paket
sudo apt install nginx mysql-server php-fpm

# Belirli versiyon
sudo apt install nginx=1.18.0-0ubuntu1

# Paket kaldır
sudo apt remove nginx

# Paket ve ayarları sil
sudo apt purge nginx

# Gereksiz paketleri temizle
sudo apt autoremove

# İndirilen cache'i temizle
sudo apt clean
```

---

### Arama ve Bilgi

```bash
# Paket ara
apt search nginx

# Cache'te ara (daha hızlı)
apt-cache search nginx

# Paket bilgisi
apt show nginx

# Hangi dosyaları kuruyor?
apt-file list nginx

# Kurulu mu?
dpkg -l | grep nginx

# Hangi paket bu dosyayı kurdu?
dpkg -S /usr/bin/nginx
```

---

### Repository Yönetimi

```bash
# Repository listesi
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Repository ekle
sudo add-apt-repository ppa:ondrej/php
sudo apt update

# Repository sil
sudo add-apt-repository --remove ppa:ondrej/php

# Manuel repository ekle
echo "deb http://repo.example.com/ubuntu focal main" | \
  sudo tee /etc/apt/sources.list.d/example.list

# GPG anahtarı ekle
curl -fsSL https://example.com/key.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/example.gpg
```

---

### Güncelleme Stratejileri

```bash
# Güvenli güncelleme (önerilir)
sudo apt update && sudo apt upgrade -y

# Otomatik evet de
sudo apt update && sudo apt upgrade -y

# Dry-run (sadece simülasyon)
sudo apt upgrade --dry-run

# Belirli paketi güncelle
sudo apt install --only-upgrade nginx
```

---

## DNF/YUM (Red Hat/Rocky/Fedora)

### Temel Komutlar

```bash
# Sistemi güncelle
sudo dnf update
# veya (eski)
sudo yum update

# Paket kur
sudo dnf install nginx

# Grup kur (birden fazla paket)
sudo dnf groupinstall "Development Tools"

# Paket kaldır
sudo dnf remove nginx

# Paket ara
dnf search nginx

# Paket bilgisi
dnf info nginx

# Hangi paket bu dosyayı sağlar?
dnf provides /usr/bin/nginx

# Kurulu paketler
dnf list installed

# Güncelleme var mı?
dnf check-update

# Gereksiz bağımlılıkları kaldır
sudo dnf autoremove

# Cache temizle
sudo dnf clean all
```

---

### Repository Yönetimi (DNF/YUM)

```bash
# Repository listesi
dnf repolist

# Repository ekle
sudo dnf config-manager --add-repo https://example.com/repo

# EPEL repository (Extra Packages for Enterprise Linux)
sudo dnf install epel-release

# Repository devre dışı bırak
sudo dnf config-manager --disable repo-name

# Repository aktifleştir
sudo dnf config-manager --enable repo-name
```

---

## Pacman (Arch Linux)

```bash
# Sistemi güncelle
sudo pacman -Syu

# Paket kur
sudo pacman -S nginx

# Paket kaldır
sudo pacman -R nginx

# Paket ve bağımlılıklarını kaldır
sudo pacman -Rs nginx

# Paket ara
pacman -Ss nginx

# Paket bilgisi
pacman -Si nginx

# Kurulu mu?
pacman -Q nginx

# Yetim paketleri kaldır
sudo pacman -Rns $(pacman -Qtdq)

# Cache temizle
sudo pacman -Sc
```

---

## Snap

**Canonical'ın evrensel paket sistemi.**

```bash
# Snap kur (Ubuntu'da default)
sudo apt install snapd

# Paket kur
sudo snap install nginx

# Paketleri listele
snap list

# Paket ara
snap find nginx

# Paket bilgisi
snap info nginx

# Paket güncelle
sudo snap refresh nginx

# Tüm paketleri güncelle
sudo snap refresh

# Paket kaldır
sudo snap remove nginx

# Channel değiştir
sudo snap switch nginx --channel=stable
sudo snap refresh nginx
```

**Avantajlar:**
- Tüm dağıtımlarda çalışır
- Otomatik güncelleme
- İzole çalışır (sandbox)

**Dezavantajlar:**
- Daha büyük boyut
- Başlangıç yavaş olabilir

---

## Flatpak

**Flathub'ın evrensel paket sistemi.**

```bash
# Flatpak kur
sudo apt install flatpak

# Flathub ekle
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Uygulama kur
flatpak install flathub org.gimp.GIMP

# Çalıştır
flatpak run org.gimp.GIMP

# Listele
flatpak list

# Güncelle
flatpak update

# Kaldır
flatpak uninstall org.gimp.GIMP

# Gereksizleri temizle
flatpak uninstall --unused
```

---

## AppImage

**Portable uygulamalar** (Windows .exe gibi).

```bash
# İndir
wget https://example.com/app.AppImage

# Çalıştırılabilir yap
chmod +x app.AppImage

# Çalıştır
./app.AppImage

# Sistem entegrasyonu
./app.AppImage --appimage-extract
```

**Avantajlar:**
- Kurulum gerektirmez
- Portable
- Bağımlılık sorunu yok

---

## Binary Kurulumları

### .deb Dosyası (Debian/Ubuntu)

```bash
# .deb dosyası kur
sudo dpkg -i paket.deb

# Bağımlalık hatası çözme
sudo apt install -f

# veya direkt apt ile
sudo apt install ./paket.deb
```

---

### .rpm Dosyası (Red Hat/Rocky/Fedora)

```bash
# .rpm dosyası kur
sudo rpm -ivh paket.rpm

# Zaten kuruluysa güncelle
sudo rpm -Uvh paket.rpm

# Bağımlılıkları hallederek kur
sudo dnf install paket.rpm
```

---

## Kaynak Koddan Kurulum

```bash
# Gerekli araçlar
sudo apt install build-essential

# Kaynak kodu indir ve aç
wget https://example.com/program-1.0.tar.gz
tar -xzvf program-1.0.tar.gz
cd program-1.0

# Derleme ve kurulum
./configure
make
sudo make install

# Kaldırma (bazen mümkün)
sudo make uninstall
```

**configure seçenekleri:**
```bash
# Kurulum dizini belirt
./configure --prefix=/opt/program

# Yardım
./configure --help
```

---

## PPA (Personal Package Archive) - Ubuntu

```bash
# PPA ekle
sudo add-apt-repository ppa:ondrej/php
sudo apt update

# PPA'dan kur
sudo apt install php8.2

# PPA kaldır
sudo add-apt-repository --remove ppa:ondrej/php
sudo apt update
```

**Popüler PPA'lar:**
```bash
# PHP (Ondřej Surý)
sudo add-apt-repository ppa:ondrej/php

# Nginx mainline
sudo add-apt-repository ppa:nginx/development

# Git son versiyon
sudo add-apt-repository ppa:git-core/ppa
```

---

## Pinning - Versiyon Kilitleme

Belirli paketin güncellenmesini engelle.

**APT Pinning:**

```bash
# /etc/apt/preferences.d/nginx oluştur
sudo tee /etc/apt/preferences.d/nginx << EOF
Package: nginx
Pin: version 1.18.0-*
Pin-Priority: 1001
EOF

# Artık nginx güncellenmeye çalışılsa bile 1.18.0 kalır
```

**DNF Versionlock:**

```bash
# Plugin kur
sudo dnf install python3-dnf-plugin-versionlock

# Paketi kilitle
sudo dnf versionlock add nginx

# Kilitlenenler
dnf versionlock list

# Kilidi aç
sudo dnf versionlock delete nginx
```

---

## Otomatik Güncellemeler

### Ubuntu - Unattended Upgrades

```bash
# Kur
sudo apt install unattended-upgrades

# Yapılandır
sudo dpkg-reconfigure -plow unattended-upgrades

# Ayarlar
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

# Çalıştır (test)
sudo unattended-upgrade --dry-run --debug
```

**Önerilen ayar:**
```
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    // "${distro_id}:${distro_codename}-updates";
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Mail "admin@example.com";
Unattended-Upgrade::Automatic-Reboot "false";
```

---

### Rocky Linux - dnf-automatic

```bash
# Kur
sudo dnf install dnf-automatic

# Yapılandır
sudo nano /etc/dnf/automatic.conf

# Servisi aktifleştir
sudo systemctl enable --now dnf-automatic.timer

# Durum kontrol
sudo systemctl status dnf-automatic.timer
```

---

## Paket Sorgulama

### Kurulu Paketler

**Debian/Ubuntu:**
```bash
# Tüm kurulu paketler
dpkg -l

# Belirli paket
dpkg -l | grep nginx

# Kaç paket kurulu?
dpkg -l | wc -l

# Paket detayları
dpkg -s nginx

# Hangi dosyaları kurdu?
dpkg -L nginx
```

**Rocky/Fedora:**
```bash
# Tüm kurulu paketler
rpm -qa

# Belirli paket
rpm -qa | grep nginx

# Paket detayları
rpm -qi nginx

# Hangi dosyaları kurdu?
rpm -ql nginx

# Hangi paket bu dosyayı kurdu?
rpm -qf /usr/bin/nginx
```

---

## Bağımlılık Yönetimi

```bash
# APT - Bağımlılıkları göster
apt-cache depends nginx

# APT - Hangi paketler buna bağımlı?
apt-cache rdepends nginx

# DNF - Bağımlılıklar
dnf repoquery --requires nginx

# DNF - Ters bağımlılık
dnf repoquery --whatrequires nginx
```

---

## Gerçek Senaryolar

### LAMP Stack Kurulumu (Ubuntu)

```bash
# Apache + MySQL + PHP
sudo apt update
sudo apt install -y \
  apache2 \
  mysql-server \
  php \
  php-mysql \
  libapache2-mod-php

# Servisleri başlat
sudo systemctl start apache2
sudo systemctl start mysql
sudo systemctl enable apache2
sudo systemctl enable mysql
```

---

### LEMP Stack Kurulumu (Rocky Linux)

```bash
# Nginx + MariaDB + PHP
sudo dnf install -y \
  nginx \
  mariadb-server \
  php-fpm \
  php-mysqlnd

# Servisleri başlat
sudo systemctl start nginx mariadb php-fpm
sudo systemctl enable nginx mariadb php-fpm
```

---

### Development Tools

**Ubuntu:**
```bash
sudo apt install -y \
  build-essential \
  git \
  curl \
  wget \
  vim \
  htop \
  net-tools \
  software-properties-common \
  apt-transport-https \
  ca-certificates
```

**Rocky Linux:**
```bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y \
  git \
  curl \
  wget \
  vim \
  htop \
  net-tools
```

---

## Temizlik ve Bakım

### APT Temizlik

```bash
# İndirilen paketleri sil
sudo apt clean

# Eski paket versiyonlarını sil
sudo apt autoclean

# Gereksiz bağımlılıkları kaldır
sudo apt autoremove

# Kırık bağımlılıkları düzelt
sudo apt install -f

# Hepsi birlikte
sudo apt autoremove && sudo apt autoclean
```

---

### DNF Temizlik

```bash
# Cache temizle
sudo dnf clean all

# Gereksiz paketleri kaldır
sudo dnf autoremove

# Yetim paketler
sudo dnf repoquery --unneeded
```

---

### Disk Alan Kontrolü

```bash
# APT cache boyutu
du -sh /var/cache/apt/archives

# Log boyutu
du -sh /var/log

# Eski kerneller kaldır (Ubuntu)
sudo apt autoremove --purge
```

---

## Best Practices

### ✅ Yapılması Gerekenler

1. **Düzenli güncelle**
   ```bash
   # Haftalık
   sudo apt update && sudo apt upgrade
   ```

2. **Güvenlik güncellemelerini otomatikleştir**
   ```bash
   sudo apt install unattended-upgrades
   ```

3. **Snapshot al (production'da)**
   ```bash
   # LVM snapshot
   sudo lvcreate -L 5G -s -n snap /dev/vg0/root
   ```

4. **Güncelleme öncesi backup**

5. **Test ortamında dene**

---

### ❌ Yapılmaması Gerekenler

1. ❌ **dist-upgrade körü körüne çalıştırma**
   ```bash
   # Dikkatli ol, major versiyon değiştirir
   sudo apt dist-upgrade
   ```

2. ❌ **Bilinmeyen repository eklemek**
   ```bash
   # Güvenilir mi kontrol et!
   ```

3. ❌ **Production'da direkt güncelleme**
   - Önce staging'de test et

4. ❌ **Karışık repository'ler**
   ```bash
   # Ubuntu'ya Debian paketi kurma
   # Rocky'ye Ubuntu paketi kurma
   ```

---

## Sorun Giderme

### Broken Packages

**Ubuntu:**
```bash
# Kırık paketleri düzelt
sudo apt install -f
sudo dpkg --configure -a

# Paket veritabanını yeniden oluştur
sudo apt update

# Zorla yeniden kur
sudo apt install --reinstall nginx
```

**Rocky:**
```bash
# RPM veritabanını yeniden oluştur
sudo rpm --rebuilddb

# Cache temizle
sudo dnf clean all
sudo dnf makecache
```

---

### Lock Hatası

```bash
# Hata: Unable to acquire the dpkg frontend lock

# Çözüm: Kilidi kaldır
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/dpkg/lock
sudo dpkg --configure -a
```

---

### Repository Hatası

```bash
# GPG anahtarı hatası
# Anahtarı tekrar ekle
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys KEY_ID
```

---

## 🎯 Özet

| Dağıtım | Güncelle | Kur | Kaldır | Ara |
|---------|----------|-----|--------|-----|
| **Ubuntu** | `apt update && apt upgrade` | `apt install pkg` | `apt remove pkg` | `apt search pkg` |
| **Rocky** | `dnf update` | `dnf install pkg` | `dnf remove pkg` | `dnf search pkg` |
| **Arch** | `pacman -Syu` | `pacman -S pkg` | `pacman -R pkg` | `pacman -Ss pkg` |

---

## 🎓 Alıştırmalar

1. Sistemi tam güncelle
2. `nginx`, `curl`, `htop` kur
3. Kurulu tüm paketleri listele
4. `htop` paketini kaldır sonra tekrar kur
5. Bir PPA ekle (örn: Git son versiyon)
6. `snap` veya `flatpak` ile bir uygulama kur
7. Gereksiz paketleri temizle
8. Repository listesini görüntüle
9 Belirli bir paketin bağımlılıklarını listele
10. Otomatik güncelleme yapılandır

---

**Sonraki Bölüm:** [05-Kullanici-ve-Grup-Yonetimi.md](05-Kullanici-ve-Grup-Yonetimi.md) → Kullanıcılar ve yetkiler
