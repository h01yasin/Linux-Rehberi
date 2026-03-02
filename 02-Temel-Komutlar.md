# 🐧 LINUX REHBERİ — BÖLÜM 2: TEMEL KOMUTLAR

Terminal'de ustalaşmak için bilmen gereken temel komutlar.

---

## Navigasyon Komutları

### pwd (Print Working Directory)

Şu anki dizini gösterir.

```bash
pwd
# /home/yasin
```

---

### cd (Change Directory)

Dizin değiştirme.

```bash
# Home dizinine git
cd
cd ~
cd /home/yasin

# Bir üst dizin
cd ..

# İki üst dizin
cd ../..

# Root dizine git
cd /

# Önceki dizine geri dön
cd -

# Mutlak yol
cd /var/log

# Göreceli yol
cd Documents/projects
```

---

### ls (List)

Dosya/dizin listele.

```bash
# Basit liste
ls

# Detaylı (long format)
ls -l

# Gizli dosyalarla (.bashrc gibi)
ls -a

# İnsan okunabilir boyutlar
ls -lh

# Hepsi birlikte
ls -lah

# Tarihe göre sırala (en yeni üstte)
ls -lt

# Boyuta göre sırala
ls -lS

# Recursive (alt dizinler dahil)
ls -R

# Tek sütun
ls -1

# Renklendirme
ls --color=auto
```

**Çıktı açıklaması:**
```bash
ls -l
# -rw-r--r--  1 yasin yasin  1024 Jan 15 10:30 dosya.txt
# │││││││││  │   │     │      │    │           └─ Dosya adı
# │││││││││  │   │     │      │    └─ Tarih/Saat
# │││││││││  │   │     │      └─ Boyut (byte)
# │││││││││  │   │     └─ Grup
# │││││││││  │   └─ Sahip
# │││││││││  └─ Hard link sayısı
# └─────────── İzinler
```

---

## Dosya İşlemleri

### touch

Boş dosya oluştur veya tarihi güncelle.

```bash
# Boş dosya oluştur
touch dosya.txt

# Birden fazla
touch file1.txt file2.txt file3.txt

# Tarihi güncelle (dosya varsa)
touch mevcut_dosya.txt
```

---

### mkdir (Make Directory)

Dizin oluştur.

```bash
# Basit
mkdir yeni_dizin

# Birden fazla
mkdir dir1 dir2 dir3

# Alt dizinlerle (parent directories)
mkdir -p projeler/web/frontend
# projeler, web, frontend hepsini oluşturur

# İzinle birlikte
mkdir -m 755 public_dir
```

---

### cp (Copy)

Dosya/dizin kopyala.

```bash
# Dosya kopyala
cp kaynak.txt hedef.txt

# Dizine kopyala
cp dosya.txt /home/yasin/Documents/

# Dizin kopyala (recursive)
cp -r kaynak_dizin hedef_dizin

# İnteraktif (üzerine yazmadan önce sor)
cp -i dosya.txt hedef.txt

# Verbose (ne yaptığını göster)
cp -v dosya.txt hedef.txt

# İzinleri koru
cp -p dosya.txt hedef.txt

# Sembolik linkleri kopyala
cp -d link hedef/

# Tümü birlikte (archive mode)
cp -a kaynak/ hedef/
```

**Örnekler:**
```bash
# Backup al
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup

# Tüm .txt dosyalarını kopyala
cp *.txt backup/

# Dizini yedekle
cp -r /var/www/html /backup/html_$(date +%Y%m%d)
```

---

### mv (Move)

Taşı veya yeniden adlandır.

```bash
# Yeniden adlandır
mv eski.txt yeni.txt

# Taşı
mv dosya.txt /home/yasin/Documents/

# Dizin taşı
mv eski_dizin yeni_dizin

# Birden fazla dosyayı dizine taşı
mv file1.txt file2.txt file3.txt /hedef/

# İnteraktif
mv -i dosya.txt hedef.txt

# Backup oluştururken taşı
mv -b dosya.txt hedef.txt
```

---

### rm (Remove)

Dosya/dizin sil.

```bash
# Dosya sil
rm dosya.txt

# Birden fazla
rm file1.txt file2.txt

# Dizin sil (recursive)
rm -r dizin/

# Zorla sil (onay sorma)
rm -f dosya.txt

# Hem recursive hem force
rm -rf dizin/
# ⚠️ DİKKAT: Çok tehlikeli! Geri dönüşü yok!

# İnteraktif (her dosya için sor)
rm -i dosya.txt

# Verbose
rm -v dosya.txt

# Dizini boşalt ama kendini silme
rm -r dizin/*
```

**⚠️ TEHLİKELİ KOMUTLAR:**
```bash
# ASLA ÇALIŞTIRMA!
sudo rm -rf /         # Tüm sistemi siler
sudo rm -rf /*        # Aynı şey
rm -rf ~/*            # Home dizinini siler
```

---

### cat (Concatenate)

Dosya içeriğini göster.

```bash
# Dosyayı göster
cat dosya.txt

# Birden fazla dosya
cat file1.txt file2.txt

# Dosyaları birleştir
cat file1.txt file2.txt > birlesmis.txt

# Yeni dosya oluştur (Ctrl+D ile bitir)
cat > yeni.txt
Bu bir test
Buraya yazıyorum
[Ctrl+D]

# Dosyaya ekle (append)
cat >> dosya.txt
Yeni satır
[Ctrl+D]

# Satır numaraları ile
cat -n dosya.txt

# Boş satırları gösterme
cat -s dosya.txt
```

---

### less / more

Dosyayı sayfalayarak göster.

```bash
# less (modern, daha iyi)
less dosya.txt
# Space: Sonraki sayfa
# b: Önceki sayfa
# /kelime: Ara
# q: Çık

# more (eski)
more dosya.txt
# Space: Sonraki sayfa
# q: Çık

# Log takibi
less +F /var/log/syslog
# Ctrl+C: Durdur, q: Çık
```

---

### head / tail

Dosyanın başını veya sonunu göster.

```bash
# İlk 10 satır (default)
head dosya.txt

# İlk 20 satır
head -n 20 dosya.txt
head -20 dosya.txt

# İlk 50 byte
head -c 50 dosya.txt

# Son 10 satır
tail dosya.txt

# Son 50 satır
tail -n 50 dosya.txt

# Canlı takip (log için mükemmel)
tail -f /var/log/syslog
# Ctrl+C ile dur

# Son 100 satırı göster ve takip et
tail -n 100 -f /var/log/nginx/access.log
```

---

### wc (Word Count)

Satır/kelime/karakter say.

```bash
# Satır, kelime, byte
wc dosya.txt
# 10 50 350 dosya.txt

# Sadece satır sayısı
wc -l dosya.txt

# Sadece kelime
wc -w dosya.txt

# Sadece karakter
wc -c dosya.txt

# Kaç log satırı var?
wc -l /var/log/syslog
```

---

## Arama Komutları

### find

Dosya/dizin ara.

```bash
# Tüm dosyaları listele
find .

# İsme göre ara
find /home -name "dosya.txt"

# Case-insensitive
find /home -iname "DOSYA.txt"

# .txt uzantılı dosyalar
find /home -name "*.txt"

# Dizin ara
find / -type d -name "logs"

# Dosya ara
find / -type f -name "*.log"

# Son 7 günde değişenler
find /var/log -mtime -7

# 100MB'dan büyük dosyalar
find /home -size +100M

# Boş dosyalar
find /tmp -empty

# Sahip kullanıcıya göre
find /home -user yasin

# İzinlere göre
find /var -perm 777

# Ara ve sil
find /tmp -name "*.tmp" -delete

# Ara ve komut çalıştır
find /var/log -name "*.log" -exec ls -lh {} \;
```

**Gerçek örnekler:**
```bash
# Büyük dosyaları bul
find / -type f -size +1G 2>/dev/null

# Son 24 saatte değiştirilen PHP dosyaları
find /var/www -name "*.php" -mtime -1

# 777 izinli dosyaları bul (güvenlik riski)
find /var/www -type f -perm 777
```

---

### grep

Metin içinde ara.

```bash
# Dosyada kelime ara
grep "kelime" dosya.txt

# Case-insensitive
grep -i "KeLiMe" dosya.txt

# Satır numarası ile
grep -n "error" /var/log/syslog

# Recursive (tüm dosyalarda)
grep -r "TODO" /var/www/

# Sadece dosya adlarını göster
grep -rl "TODO" /var/www/

# Ters arama (içermeyenleri göster)
grep -v "success" log.txt

# Birden fazla kelime (OR)
grep -E "error|warning" log.txt
grep "error\|warning" log.txt

# Tam kelime eşleşmesi
grep -w "cat" dosya.txt
# "cat" bulur ama "category" bulamaz

# Birkaç satır önce/sonra göster
grep -A 3 "error" log.txt    # Sonraki 3 satır
grep -B 3 "error" log.txt    # Önceki 3 satır
grep -C 3 "error" log.txt    # Önce ve sonra 3'er satır

# Renklendirerek göster
grep --color=auto "error" log.txt
```

**Örnekler:**
```bash
# IP adresi ara
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log

# Hata logları
grep -i error /var/log/syslog

# Belirli bir kullanıcının işlemleri
ps aux | grep yasin

# Port dinleyenler
netstat -tulpn | grep LISTEN
```

---

### locate

Dosyayı veritabanından hızlıca ara.

```bash
# Kurulum
sudo apt install mlocate

# Veritabanını güncelle
sudo updatedb

# Ara
locate dosya.txt

# Case-insensitive
locate -i DOSYA.txt

# Sayımla
locate -c *.txt

# Sadece mevcut olanlar
locate -e dosya.txt
```

**find vs locate:**
- `find`: Yavaş ama gerçek zamanlı
- `locate`: Hızlı ama veritabanı güncel olmayabilir

---

## Dosya İçeriği

### echo

Metin yazdır veya dosyaya yaz.

```bash
# Ekrana yazdır
echo "Merhaba Dünya"

# Dosyaya yaz (üzerine)
echo "test" > dosya.txt

# Dosyaya ekle (append)
echo "yeni satır" >> dosya.txt

# Environment variable
echo $HOME
echo $USER
echo $PATH

# Yeni satır
echo -e "Satır 1\nSatır 2"

# Yeni satır olmadan
echo -n "Başlangıç"
```

---

### Yönlendirme (Redirection)

```bash
# Çıktıyı dosyaya yaz
ls -l > liste.txt

# Dosyaya ekle
ls -l >> liste.txt

# Hata çıktısını yönlendir
komut 2> hatalar.txt

# Hem çıktı hem hata
komut > output.txt 2>&1
komut &> output.txt

# Çıktıyı gizle
komut > /dev/null

# Hem çıktı hem hata gizle
komut > /dev/null 2>&1

# Dosyadan girdi al
sort < liste.txt
```

---

### Pipe (|)

Bir komutun çıktısını diğerine gönder.

```bash
# Process listesinde ara
ps aux | grep nginx

# Sırala
ls -l | sort -k5 -n

# Satır say
cat dosya.txt | wc -l

# İlk 10'u al
ls -lt | head -10

# Çoklu pipe
ps aux | grep nginx | grep -v grep | awk '{print $2}'

# Dosya sayısı
ls | wc -l

# En büyük 5 dosya
ls -lh | sort -k5 -h | tail -5
```

---

## Bilgi Komutları

### file

Dosya tipini göster.

```bash
file dosya.txt
# dosya.txt: ASCII text

file image.jpg
# image.jpg: JPEG image data

file /bin/ls
# /bin/ls: ELF 64-bit LSB executable

file script.sh
# script.sh: Bash script, ASCII text executable
```

---

### stat

Dosya hakkında detaylı bilgi.

```bash
stat dosya.txt
# Dosya: dosya.txt
# Boyut: 1024
# İzinler: 0644
# Erişim zamanı
# Değiştirilme zamanı
# vb.
```

---

### du (Disk Usage)

Dizin/dosya boyutu.

```bash
# Boyut göster
du dosya.txt

# İnsan okunabilir
du -h dosya.txt

# Özet (toplam)
du -sh /var/www

# En büyük 10 dizin
du -h /home | sort -h | tail -10

# Derinlik limiti
du -h --max-depth=1 /var
```

---

### df (Disk Free)

Disk kullanımı.

```bash
# Tüm diskler
df

# İnsan okunabilir
df -h

# Tip göster
df -T

# İnode kullanımı
df -i

# Sadece ext4 sistemler
df -t ext4
```

---

## Arşivleme ve Sıkıştırma

### tar

Arşiv oluştur/aç.

```bash
# Arşiv oluştur
tar -cvf arsiv.tar dizin/
# c: create, v: verbose, f: file

# Sıkıştırarak arşivle (gzip)
tar -czvf arsiv.tar.gz dizin/

# Sıkıştırarak arşivle (bzip2)
tar -cjvf arsiv.tar.bz2 dizin/

# Arşiv aç
tar -xvf arsiv.tar
# x: extract

# Sıkıştırılmış arşiv aç
tar -xzvf arsiv.tar.gz

# Belirli dizine aç
tar -xzvf arsiv.tar.gz -C /hedef/dizin/

# Arşiv içeriğini listele
tar -tvf arsiv.tar

# Belirli dosyayı çıkar
tar -xvf arsiv.tar dosya.txt
```

**Gerçek örnekler:**
```bash
# Website yedeği
tar -czvf website_$(date +%Y%m%d).tar.gz /var/www/html

# Backup'ı aç
tar -xzvf website_20240315.tar.gz -C /var/www/html

# Log arşivle
tar -czvf logs_$(date +%Y%m).tar.gz /var/log/*.log
```

---

### zip / unzip

```bash
# Kurulum
sudo apt install zip unzip

# Zip oluştur
zip arsiv.zip dosya1.txt dosya2.txt

# Dizini zip'le
zip -r arsiv.zip dizin/

# Zip aç
unzip arsiv.zip

# Belirli dizine aç
unzip arsiv.zip -d /hedef/

# Liste göster
unzip -l arsiv.zip
```

---

### gzip / gunzip

```bash
# Sıkıştır (orijinal silinir)
gzip dosya.txt
# dosya.txt.gz oluşur

# Aç
gunzip dosya.txt.gz

# Orijinali koru
gzip -k dosya.txt

# Dosyayı açmadan göster
zcat dosya.txt.gz
```

---

## Sistem Komutları

### history

Komut geçmişi.

```bash
# Geçmişi göster
history

# Son 20 komut
history 20

# Geçmişi ara
history | grep apt

# Son komutu tekrar çalıştır
!!

# 5. komutu çalıştır
!5

# "apt" ile başlayan son komutu
!apt

# Geçmişi temizle
history -c
```

---

### clear

Ekranı temizle.

```bash
clear
# veya Ctrl + L
```

---

### alias

Kısayol oluştur.

```bash
# Alias oluştur
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade'
alias cls='clear'

# Kullan
ll

# Alias'ları göster
alias

# Alias sil
unalias ll

# Kalıcı alias (~/.bashrc'ye ekle)
echo "alias ll='ls -lah'" >> ~/.bashrc
source ~/.bashrc
```

---

## 🎯 Özet Tablo

| Komut | Açıklama | Örnek |
|-------|----------|-------|
| `pwd` | Şu anki dizin | `pwd` |
| `cd` | Dizin değiştir | `cd /var/log` |
| `ls` | Listele | `ls -lah` |
| `mkdir` | Dizin oluştur | `mkdir test` |
| `touch` | Dosya oluştur | `touch file.txt` |
| `cp` | Kopyala | `cp a.txt b.txt` |
| `mv` | Taşı/yeniden adlandır | `mv old.txt new.txt` |
| `rm` | Sil | `rm file.txt` |
| `cat` | Göster | `cat file.txt` |
| `less` | Sayfalayarak göster | `less file.txt` |
| `head` | İlk satırlar | `head -20 file.txt` |
| `tail` | Son satırlar | `tail -f log.txt` |
| `find` | Dosya ara | `find / -name "*.log"` |
| `grep` | Metin ara | `grep "error" log.txt` |
| `tar` | Arşivle | `tar -czvf backup.tar.gz dir/` |

---

## 🎓 Alıştırmalar

1. Home dizininde `test` klasörü oluştur, içine 5 boş dosya ekle
2. Bu dosyaların hepsini `backup` klasörüne kopyala
3. `*.txt` dosyalarını bul ve say
4. `/var/log/syslog` dosyasında "error" kelimesini ara
5. En büyük 10 dosyayı `/home` dizininde bul
6. Bir dizini tar.gz olarak arşivle ve sonra aç
7. `ps aux` çıktısında belirli bir süreci grep ile bul
8. `.bashrc` dosyasına 3 kullanışlı alias ekle
9. `find` ve `rm` komutlarını birleştirerek `.tmp` dosyalarını sil
10. `tail -f` ile canlı log takibi yap

---

**Sonraki Bölüm:** [03-Dosya-Sistemi-ve-Izinler.md](03-Dosya-Sistemi-ve-Izinler.md) → İzinler ve dosya sistemi
