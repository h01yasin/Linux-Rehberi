# 🐧 LINUX REHBERİ — Sıfırdan İleri Seviye Kapsamlı Kılavuz

Linux sistem yönetimi ve DevOps için hazırlanmış tam kapsamlı Türkçe rehber.

---

## 📚 İçindekiler

### 🎯 Temel Bölümler (Başlangıç)

| # | Dosya | Konu | İçerik |
|---|-------|------|--------|
| 1 | [01-Giris-ve-Dagitimlar.md](01-Giris-ve-Dagitimlar.md) | Linux'a Giriş | Linux nedir, Dağıtımlar, Kurulum, İlk adımlar |
| 2 | [02-Temel-Komutlar.md](02-Temel-Komutlar.md) | Temel Komutlar | ls, cd, cp, mv, rm, cat, grep, find, navigation |
| 3 | [03-Dosya-Sistemi-ve-Izinler.md](03-Dosya-Sistemi-ve-Izinler.md) | Dosya Sistemi | chmod, chown, permissions, FHS, inodes |
| 4 | [04-Paket-Yonetimi.md](04-Paket-Yonetimi.md) | Paket Yönetimi | apt, yum, dnf, pacman, snap, flatpak |
| 5 | [05-Kullanici-ve-Grup-Yonetimi.md](05-Kullanici-ve-Grup-Yonetimi.md) | Kullanıcı/Grup | useradd, passwd, sudo, groups, /etc/passwd |
| 6 | [06-Surec-Yonetimi.md](06-Surec-Yonetimi.md) | Süreç Yönetimi | ps, top, htop, kill, jobs, systemctl, cron |

### 🚀 Orta Seviye Bölümler

| # | Dosya | Konu | İçerik |
|---|-------|------|--------|
| 7 | [07-Ag-Yonetimi.md](07-Ag-Yonetimi.md) | Ağ (Network) | ip, netstat, ss, firewall, routing, DNS |
| 8 | [08-Shell-Scripting.md](08-Shell-Scripting.md) | Bash Scripting | Variables, loops, functions, debugging |
| 9 | [09-Metin-Isleme.md](09-Metin-Isleme.md) | Metin Araçları | sed, awk, grep, cut, tr, vim/nano |
| 10 | [10-Servis-Yonetimi.md](10-Servis-Yonetimi.md) | Systemd | systemctl, units, targets, timers |

### 🔒 İleri Seviye (Sistem Yöneticisi)

| # | Dosya | Konu | İçerik |
|---|-------|------|--------|
| 11 | [11-Sistem-Guvenligi.md](11-Sistem-Guvenligi.md) | Güvenlik | SSH, Firewall, SELinux, AppArmor, fail2ban |
| 12 | [12-Log-ve-Troubleshooting.md](12-Log-ve-Troubleshooting.md) | Log & Debug | journalctl, syslog, log analizi, sorun giderme |
| 13 | [13-Disk-ve-Storage.md](13-Disk-ve-Storage.md) | Disk Yönetimi | fdisk, LVM, RAID, mount, fstab, quotas |
| 14 | [14-Performans-Monitoring.md](14-Performans-Monitoring.md) | Performance | top, iotop, sar, vmstat, tuning |
| 15 | [15-Backup-ve-Recovery.md](15-Backup-ve-Recovery.md) | Yedekleme | rsync, tar, dd, backup stratejileri |

### 🚀 DevOps Bölümleri

| # | Dosya | Konu | İçerik |
|---|-------|------|--------|
| 16 | [16-Web-Sunucu-Yonetimi.md](16-Web-Sunucu-Yonetimi.md) | Web Servers | Apache, Nginx, SSL, reverse proxy |
| 17 | [17-Database-Yonetimi.md](17-Database-Yonetimi.md) | Veritabanı | MySQL, PostgreSQL, Redis, yedekleme |
| 18 | [18-Automation-Ansible.md](18-Automation-Ansible.md) | Otomasyon | Ansible basics, playbooks, roles |
| 19 | [19-Container-ve-Virtualization.md](19-Container-ve-Virtualization.md) | Containers | Docker, Podman, KVM, QEMU |
| 20 | [20-CI-CD-ve-Git.md](20-CI-CD-ve-Git.md) | CI/CD & Git | Git workflow, Jenkins, pipelines |

---

## 🚀 Hızlı Başlangıç

### Linux Dağıtımı Seç ve Kur

**Başlangıç için önerilen:**
- **Ubuntu 24.04 LTS** - En popüler, kolay
- **Rocky Linux / AlmaLinux** - Enterprise (RHEL alternatifi)
- **Debian** - Stabil, güvenilir

**Test için:**
- VirtualBox veya VMware ile sanal makine
- WSL2 (Windows Subsystem for Linux)
- Cloud (AWS, Azure, DigitalOcean)

### İlk Komutlar

```bash
# Sistem bilgisi
uname -a
cat /etc/os-release

# Güncellemeler
sudo apt update && sudo apt upgrade -y  # Debian/Ubuntu
sudo dnf update -y                       # Fedora/Rocky

# Yardım
man komut-adi
komut-adi --help
```

---

## 🗺️ Öğrenme Yolu

```
Başlangıç (1-6)         Orta (7-10)            İleri (11-15)         DevOps (16-20)
─────────────────────────────────────────────────────────────────────────────────
├─ Terminal kullanımı   ├─ Ağ yapılandırması   ├─ Güvenlik          ├─ Web sunucular
├─ Dosya işlemleri      ├─ Bash scripting      ├─ Log analizi       ├─ Veritabanları
├─ İzinler              ├─ Metin işleme        ├─ Disk yönetimi     ├─ Ansible
├─ Paket yönetimi       └─ Servisler           ├─ Performans        ├─ Containers
├─ Kullanıcı yönetimi                          └─ Backup            └─ CI/CD
└─ Süreç kontrolü
```

**Tavsiye edilen öğrenme sırası:**

1. 📘 **Bölüm 1-3:** Linux'u tanı, terminal'de rahat ol, dosya sistemi
2. 📗 **Bölüm 4-6:** Paket kur, kullanıcı yönet, süreçleri kontrol et
3. 📙 **Bölüm 7-10:** Ağ, scripting, servisler
4. 📕 **Bölüm 11-15:** Güvenlik, troubleshooting, performans (Sistem Yöneticisi)
5. 🚀 **Bölüm 16-20:** Web, DB, automation (DevOps)

---

## 💡 Hızlı Referans

### En Çok Kullanılan Komutlar

```bash
# Dosya işlemleri
ls -lah                    # Detaylı liste
cd /var/log                # Dizin değiştir
cp -r kaynak hedef         # Kopyala (recursive)
mv eski yeni               # Taşı/Yeniden adlandır
rm -rf dizin               # Sil (dikkatli!)
find / -name "dosya"       # Dosya ara

# Metin görüntüleme
cat dosya.txt              # Tüm içeriği göster
less dosya.txt             # Sayfa sayfa göster
head -n 20 dosya           # İlk 20 satır
tail -f /var/log/syslog    # Canlı log takibi
grep "hata" dosya          # Metin ara

# Sistem bilgisi
df -h                      # Disk kullanımı
free -h                    # RAM kullanımı
top                        # CPU/Memory canlı
ps aux                     # Tüm süreçler
netstat -tulpn             # Açık portlar

# Kullanıcı/İzin
sudo su                    # Root ol
chmod 755 dosya            # İzin değiştir
chown user:group dosya     # Sahiplik değiştir
passwd kullanici           # Şifre değiştir

# Servis yönetimi
systemctl status nginx     # Servis durumu
systemctl start nginx      # Başlat
systemctl enable nginx     # Açılışta başlat
journalctl -xe             # Sistem logları

# Ağ
ip addr show               # IP adresleri
ping google.com            # Bağlantı test
curl ifconfig.me           # Public IP
ss -tulpn                  # Açık portlar
```

---

## 🎯 Rehber İçeriği

**Bu rehberde:**
- 📚 **20 detaylı bölüm** - Temellerden ileri seviyeye
- 📝 **6000+ satır** kod ve açıklama
- 🎓 **200+ pratik alıştırma** - Her bölümde hands-on
- 💻 **Gerçek dünya senaryoları** - Production örnekleri
- 🔒 **Güvenlik best practices** - Enterprise standartları
- 🚀 **DevOps araçları** - Modern teknolojiler
- 📊 **Sistem yönetimi** - Monitoring, backup, troubleshooting
- ☁️ **Cloud-ready** - AWS, Azure örnekleri

**Hedef Kitle:**
- ✅ Linux'a yeni başlayanlar
- ✅ Sistem yöneticisi olmak isteyenler
- ✅ DevOps mühendisliği yolunda olanlar
- ✅ Server yönetimi öğrenmek isteyenler
- ✅ Linux sertifikalarına hazırlananlar (LPIC, RHCSA)

---

## 📖 Nasıl Kullanılır?

### Yeni Başlayanlar
1. **Bölüm 1-6'yı sırayla** takip et
2. Her bölümdeki **alıştırmaları yap**
3. Kendi Linux sisteminde **pratik yap**
4. Hata yaptıkça öğren (VM kullan, çekinme!)

### Sistem Yöneticisi Yolu
1. Temelleri hızlı geç (1-6)
2. **Bölüm 11-15'e** odaklan
3. Production senaryolarını test et
4. Kendi automation scriptleri yaz

### DevOps Yolu
1. Temel Linux bilgisi edin (1-10)
2. **Bölüm 16-20'ye** geç
3. Docker + Kubernetes öğren
4. CI/CD pipeline kur

---

## 🛠️ Gereksinimler

### Yazılım
- Linux dağıtımı (Ubuntu, Rocky, Debian)
- Terminal erişimi
- Sudo yetkisi

### Önerilen Araçlar
- **Terminal emulator:** GNOME Terminal, Terminator, tmux
- **Text editor:** vim, nano, VS Code
- **SSH client:** OpenSSH, PuTTY (Windows)

### Lab Ortamı
- VirtualBox veya VMware
- En az 2 GB RAM
- 20 GB disk alanı
- İnternet bağlantısı

---

## 📝 Notlar

- ✅ **Tüm komutlar test edilmiştir**
- ✅ **Ubuntu 24.04 LTS** ve **Rocky Linux 9** üzerinde doğrulandı
- ✅ Her bölümde **gerçek senaryolar**
- ✅ **Güvenlik** vurgusu her yerde
- ✅ Production **best practices**
- ✅ Cloud ve **DevOps** odaklı

**⚠️ Uyarı:** Root yetkileri gerektiren komutlarda dikkatli olun. Test ortamında çalışın!

---

## 🤝 Katkıda Bulunma

Pull request ve issue'lar memnuniyetle karşılanır!

**Katkı yapabilirsiniz:**
- Yeni örnekler ekleyerek
- Hataları düzelterek
- Daha iyi açıklamalar yazarak
- Yeni bölümler önererek

---

## 📚 Ek Kaynaklar

- [Linux Documentation Project](https://tldp.org/)
- [ArchWiki](https://wiki.archlinux.org/) - Best Linux wiki
- [Ubuntu Docs](https://help.ubuntu.com/)
- [Red Hat Documentation](https://access.redhat.com/documentation/)

---

## 🎓 Sertifikalar

Bu rehber şu sertifikalara hazırlanmaya yardımcı olabilir:
- **LPIC-1** (Linux Professional Institute)
- **RHCSA** (Red Hat Certified System Administrator)
- **Linux Foundation Certified SysAdmin**
- **CompTIA Linux+**

---

## 📜 Lisans

Bu rehber eğitim amaçlı hazırlanmıştır ve açık kaynaklıdır.

---

**Son güncelleme:** Mart 2026  
**Desteklenen dağıtımlar:** Ubuntu 24.04, Debian 12, Rocky Linux 9, Fedora 40

🐧 **Mutlu Linux yolculukları!**
