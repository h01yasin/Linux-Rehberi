# 🐧 LINUX REHBERİ — BÖLÜM 1: GİRİŞ VE DAĞITIMLAR

---

## Linux Nedir?

**Linux**, açık kaynaklı (open source) bir işletim sistemi çekirdeğidir (kernel). 1991 yılında Linus Torvalds tarafından geliştirilmeye başlanmıştır.

**Neden Linux?**
- ✅ **Ücretsiz ve açık kaynak**
- ✅ **Güvenli** (virüs/malware riski düşük)
- ✅ **Stabil** (günler/aylar çalışır, reboot gerektirmez)
- ✅ **Hafif** (eski donanımlarda bile çalışır)
- ✅ **Özelleştirilebilir** (tamamen kontrolünüzde)
- ✅ **Sunucularda dominant** (%90+ web sunucuları)
- ✅ **DevOps/DevTools** için standart
- ✅ **Cloud altyapısı** (AWS, Azure, GCP)

---

## Linux vs Windows vs macOS

| Özellik | Linux | Windows | macOS |
|---------|-------|---------|-------|
| **Lisans** | Ücretsiz, Açık Kaynak | Ücretli | Ücretli (Mac ile gelir) |
| **Güvenlik** | Çok yüksek | Orta | Yüksek |
| **Özelleştirme** | Sınırsız | Sınırlı | Çok sınırlı |
| **Sunucu Kullanımı** | +++++ | ++ | + |
| **Gaming** | ++ | +++++ | +++ |
| **Yazılım Desteği** | Orta | Çok yüksek | Yüksek |
| **Öğrenme Eğrisi** | Orta-Zor | Kolay | Kolay-Orta |

---

## Linux Dağıtımları (Distributions)

Linux sadece bir çekirdektir. Farklı firmalar/topluluklar bu çekirdeğe yazılımlar ekleyerek **dağıtım** (distro) yapar.

### 🟢 Başlangıç İçin En İyiler

#### **Ubuntu**
- **En popüler** desktop Linux
- Kolay kurulum ve kullanım
- Geniş topluluk desteği
- 6 ayda bir yeni versiyon (LTS: 2 yılda bir, 5 yıl destek)
- **Kullanım:** Desktop, server, cloud

```bash
# Versiyon kontrolü
lsb_release -a
# Ubuntu 24.04 LTS
```

#### **Linux Mint**
- Ubuntu tabanlı
- Windows benzeri arayüz
- Yeni başlayanlar için perfect
- **Kullanım:** Desktop

#### **Pop!_OS**
- Ubuntu tabanlı
- System76 tarafından geliştiriliyor
- Gaming ve development odaklı
- Modern görünüm

---

### 🔴 Enterprise / Server

#### **Red Hat Enterprise Linux (RHEL)**
- Ücretli, enterprise destek
- Çok stabil
- **Kullanım:** Enterprise sunucular

#### **Rocky Linux / AlmaLinux**
- RHEL'in ücretsiz klonları
- CentOS'un yerine geçtiler
- Production sunucular için ideal

```bash
# Rocky Linux versiyon
cat /etc/rocky-release
# Rocky Linux release 9.3
```

#### **Debian**
- Çok stabil ve güvenilir
- Ubuntu'nun temeli
- **Kullanım:** Server

---

### ⚡ İleri Seviye

#### **Arch Linux**
- Rolling release (sürekli güncel)
- Minimal kurulum (kendin inşa et)
- En güncel paketler
- **Kullanım:** İleri seviye kullanıcılar

#### **Fedora**
- Red Hat tarafından destekleniyor
- Yeni teknolojileri ilk deneyen
- **Kullanım:** Desktop, development

#### **Gentoo**
- Her şeyi kaynak koddan derlersin
- Maksimum optimizasyon
- **Kullanım:** Extreme geeks

---

### 🐳 Özel Amaçlı

#### **Kali Linux**
- Penetration testing
- Hacking araçları
- **Kullanım:** Güvenlik testleri

#### **Proxmox**
- Virtualization platform
- KVM + Containers
- **Kullanım:** Homelab, datacenter

---

## Hangisini Seçmeliyim?

### Yeni başlıyorsan:
→ **Ubuntu 24.04 LTS** veya **Linux Mint**

### Server kuracaksan:
→ **Ubuntu Server** veya **Rocky Linux**

### DevOps/Cloud:
→ **Ubuntu** veya **Debian**

### Öğrenmek istiyorsan:
→ **Arch Linux** (zor ama çok öğretici)

---

## Linux Kurulum

### Seçenek 1: Dual Boot (Windows ile beraber)

**Adımlar:**
1. Ubuntu ISO indir: https://ubuntu.com/download
2. Rufus ile bootable USB oluştur
3. BIOS'tan USB'den boot et
4. "Install Ubuntu" → "Install alongside Windows"
5. Diski böl (örn: 50GB Linux)
6. Kullanıcı oluştur ve kur

⚠️ **Dikkat:** Yedek al!

---

### Seçenek 2: VirtualBox (Güvenli, önerilir)

```bash
# VirtualBox kur
# Windows: https://www.virtualbox.org/

# Yeni VM oluştur:
- Type: Linux
- Version: Ubuntu (64-bit)
- RAM: 4 GB
- Disk: 20-40 GB (dynamic)

# Ubuntu ISO'yu VM'e tak ve kur
```

---

### Seçenek 3: WSL2 (Windows içinde Linux)

**Windows 10/11'de:**

```powershell
# PowerShell (Admin)
wsl --install

# Ubuntu kur
wsl --install -d Ubuntu-24.04

# Başlat
wsl
```

**Avantajlar:**
- Windows içinde tam Linux
- Dosya sistemi paylaşımlı
- Hafif, hızlı

---

### Seçenek 4: Cloud (AWS, Azure, DigitalOcean)

```bash
# DigitalOcean Droplet oluştur
- Ubuntu 24.04 LTS
- $6/ay (1GB RAM)
- SSH ile bağlan
```

---

## İlk Açılış - Ubuntu Desktop

### Terminal Açma

**Kısayol:** `Ctrl + Alt + T`

Terminal = Windows CMD/PowerShell benzeri

```bash
# İlk komutun
whoami
# Kullanıcı adını gösterir

pwd
# Şu anki dizini gösterir

ls
# Dosyaları listeler
```

---

## Temel Kavramlar

### 1. Root Kullanıcısı

Linux'ta **en yetkili** kullanıcı `root`'tur (Windows'taki Administrator gibi).

```bash
# Normal kullanıcı
whoami
# yasin

# Root ol (DIKKATLI!)
sudo su
whoami
# root

# Çık
exit
```

**⚠️ Root kullanırken ÇOK DİKKATLİ OL!** Sistemin her yerine erişebilirsin.

---

### 2. Sudo Komutu

**Sudo** = "SuperUser DO" → Geçici olarak root yetkisi al

```bash
# Normal kullanıcı yazamaz
cp /etc/hosts /etc/hosts.backup
# Permission denied

# Sudo ile yapabilirsin
sudo cp /etc/hosts /etc/hosts.backup
# Şifre ister, sonra çalışır
```

**Sudo kullanırken:**
- Sadece o komut root yetkisiyle çalışır
- Şifreni sorar (her 15 dakikada bir)
- Güvenli (her işlem loglanır)

---

### 3. Dosya Sistemi Hiyerarşisi

Linux'ta her şey **tek bir ağaç yapısı**nda:

```
/                   (Root directory - en üst)
├── /home           (Kullanıcı ev dizinleri)
│   ├── /home/yasin
│   └── /home/ali
├── /etc            (Yapılandırma dosyaları)
├── /var            (Değişken veriler: loglar, cache)
├── /usr            (Kullanıcı programları)
├── /bin            (Temel komutlar: ls, cp, mv)
├── /sbin           (Sistem komutları: reboot, fsck)
├── /tmp            (Geçici dosyalar)
├── /opt            (Opsiyonel yazılımlar)
└── /root           (Root kullanıcısının evi)
```

**Windows'tan fark:**
- `C:\` yok, her şey `/` altında
- `D:\` gibi sürücüler yok, mount edilir:  `/mnt/usb`

---

### 4. Paket Yöneticisi

Linux'ta program kurmak:

```bash
# Ubuntu/Debian - APT
sudo apt update                 # Paket listesini güncelle
sudo apt install htop           # htop programını kur
sudo apt remove htop            # Kaldır
sudo apt upgrade                # Tüm paketleri güncelle

# Rocky/RHEL - DNF/YUM
sudo dnf install htop
sudo dnf update
```

**Windows'tan fark:**
- `.exe` çalıştırmazsın
- Merkezi bir "app store" gibi (ama terminal'den)
- Tüm bağımlılıkları otomatik halleder

---

### 5. Paket vs Binary vs Source

**Paket:**
```bash
sudo apt install nginx
# Hazır, derlenmiş, bağımlılıklar dahil
```

**Binary:**
```bash
wget https://example.com/program
chmod +x program
./program
```

**Source (Kaynak kod):**
```bash
git clone https://github.com/user/program
cd program
./configure
make
sudo make install
```

---

## İlk Yapılandırmalar

### 1. Sistemi Güncelle

```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# Rocky/Fedora
sudo dnf update -y

# Sistem yeniden başlat (gerekirse)
sudo reboot
```

---

### 2. Kullanıcı Bilgilerini Ayarla

```bash
# Hostname değiştir
sudo hostnamectl set-hostname myserver

# Timezone ayarla
sudo timedatectl set-timezone Europe/Istanbul
timedatectl

# Locale (dil)
locale
```

---

### 3. Gerekli Araçlar Kur

```bash
# Temel araçlar
sudo apt install -y \
  curl \
  wget \
  git \
  vim \
  htop \
  net-tools \
  build-essential

# Doğrula
git --version
curl --version
```

---

## Terminal Temelleri

### Komut Yapısı

```bash
komut [seçenekler] [argümanlar]

ls -l /home
│   │   └─ argüman (hedef)
│   └─ seçenek (long format)
└─ komut
```

### Yardım Alma

```bash
# Man pages (manual)
man ls
# Detaylı açıklama, q ile çık

# --help
ls --help
# Kısa yardım

# Which (komutun yolu)
which ls
# /usr/bin/ls
```

---

### Kısayollar

| Kısayol | Açıklama |
|---------|----------|
| `Ctrl + C` | Çalışan komutu durdur |
| `Ctrl + D` | Terminal'den çık |
| `Ctrl + L` | Ekranı temizle (`clear` gibi) |
| `Ctrl + A` | Satır başına git |
| `Ctrl + E` | Satır sonuna git |
| `Ctrl + U` | Satırı sil |
| `Tab` | Otomatik tamamlama |
| `↑ ↓` | Komut geçmişi |

---

### History (Geçmiş)

```bash
# Son komutları göster
history

# Son komutu tekrar çalıştır
!!

# 5. komutu çalıştır
!5

# "apt" ile başlayan son komutu çalıştır
!apt

# Geçmişi temizle
history -c
```

---

## Dosya İzinleri Giriş

```bash
ls -l dosya.txt
# -rw-r--r-- 1 yasin yasin 1024 Jan 1 12:00 dosya.txt
# │││││││││
# │││││││└┴ Diğerleri (other): r (okuma)
# │││││└┴── Grup (group): r (okuma)
# ││││└──── Sahip (owner): rw (okuma-yazma)
# │└┴─────── Hard link sayısı
# └──────── Dosya tipi: - (normal dosya)
```

**İzin türleri:**
- `r` = read (okuma) - 4
- `w` = write (yazma) - 2
- `x` = execute (çalıştırma) - 1

---

## Sistem Bilgisi Komutları

```bash
# İşletim sistemi
uname -a
cat /etc/os-release

# CPU bilgisi
lscpu
cat /proc/cpuinfo | head

# RAM
free -h
cat /proc/meminfo | head

# Disk
df -h               # Bağlı diskler
lsblk               # Tüm diskler

# Ağ
ip addr show        # IP adresleri
hostname -I         # Local IP
```

---

## GUI vs CLI

### GUI (Graphical User Interface)
- Pencereler, fare, ikonlar
- Ubuntu Desktop, GNOME
- Kolay ama sınırlı

### CLI (Command Line Interface)
- Terminal, komutlar
- Güçlü, esnek, hızlı
- Server'larda zorunlu
- SSH ile uzaktan erişim

**Sistem yöneticisi olarak %90 CLI kullanacaksın!**

---

## SSH ile Uzak Bağlantı

### SSH Nedir?

**Secure Shell** - Şifreli uzak bağlantı

```bash
# Server'a bağlan
ssh kullanici@192.168.1.100

# Port belirt
ssh -p 2222 kullanici@server.com

# Key ile bağlan
ssh -i ~/.ssh/id_rsa kullanici@server
```

### İlk SSH Kurulumu

```bash
# SSH server kur (sunucuda)
sudo apt install openssh-server
sudo systemctl start sshd
sudo systemctl enable sshd

# Durumu kontrol et
sudo systemctl status sshd

# Firewall'da SSH'yi aç
sudo ufw allow 22/tcp
```

---

## 🎯 Özet

Bu bölümde öğrendiklerimiz:

✅ Linux nedir, neden önemli  
✅ Popüler dağıtımlar (Ubuntu, Rocky, Debian)  
✅ Linux kurulum yöntemleri  
✅ Temel kavramlar (root, sudo, dosya sistemi)  
✅ İlk yapılandırmalar  
✅ Terminal temelleri  
✅ SSH ile uzak erişim  

---

## 🎓 Alıştırmalar

1. **Linux VM kur** VMware veya VirtualBox ile
2. Ubuntu 24.04 LTS veya başka bir dağıtım dene
3. Terminal aç ve bu komutları çalıştır:
   ```bash
   whoami
   pwd
   ls
   cd /etc
   cat /etc/os-release
   ```
4. Sistemi güncelle: `sudo apt update && sudo apt upgrade`
5. `htop` kur ve CPU/RAM kullanımı gör
6. SSH yükle ve başka bir makineden bağlan
7. 3 farklı Linux dağıtımını (Ubuntu, Fedora, Debian) VM'de kur ve karşılaştır
8. WSL2 kur (Windows kullanıyorsan)

---

**Sonraki Bölüm:** [02-Temel-Komutlar.md](02-Temel-Komutlar.md) → Linux terminal'de ustalaş!
