# 🐧 LINUX REHBERİ — BÖLÜM 9: TEXT İŞLEME VE PARSING

grep, sed, awk ve diğer text işleme araçları ile log analizi, data parsing.

---

## grep - Pattern Matching

### Basit Arama

```bash
# Dosyada ara
grep "pattern" file.txt

# Case-insensitive
grep -i "pattern" file.txt

# Tam kelime
grep -w "word" file.txt

# Satır numarası
grep -n "pattern" file.txt

# Eşleşmeyen satırlar
grep -v "pattern" file.txt

# Birden fazla dosyada
grep "pattern" *.txt

# Recursive (dizinlerde)
grep -r "pattern" /path/to/dir

# Dosya adı göster
grep -l "pattern" *.txt

# Sadece sayım
grep -c "pattern" file.txt
```

---

### Gelişmiş Örnekler

```bash
# IP adresi ara
grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' file.txt

# Email arama
grep -E '[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' file.txt

# Boş satırları gösterme
grep -v '^$' file.txt

# Yorum satırlarını gösterme
grep -v '^#' config.conf

# Context göster
grep -A 3 "error" log.txt  # Sonraki 3 satır
grep -B 3 "error" log.txt  # Önceki 3 satır
grep -C 3 "error" log.txt  # Her iki yön 3 satır

# Birden fazla pattern
grep -e "error" -e "warning" -e "critical" log.txt

# Pattern dosyadan oku
grep -f patterns.txt log.txt

# Binary dosyalarda ara
grep -a "text" binary.file
```

---

### Regular Expression Modes

```bash
# Basic regex (default)
grep 'pattern' file

# Extended regex (-E veya egrep)
grep -E 'pattern1|pattern2' file
egrep 'pattern1|pattern2' file

# Perl regex (-P)
grep -P '\d{3}-\d{4}' file

# Fixed strings (-F veya fgrep)
grep -F 'literal.string' file  # . literal olur
fgrep 'literal.string' file
```

---

### Pratik grep Kullanımları

```bash
# Log'da hata ara
grep -i error /var/log/syslog

# Aktif kullanıcılar
who | grep yasin

# Process ara
ps aux | grep nginx

# Port dinleyen processler
sudo ss -tulpn | grep LISTEN

# Başarısız SSH girişleri
sudo grep "Failed password" /var/log/auth.log

# Son 100 satırda hata ara
tail -100 /var/log/app.log | grep -i error
```

---

## sed - Stream Editor

### Basit İkame (Substitute)

```bash
# İlk eşleşmeyi değiştir
sed 's/old/new/' file.txt

# Tüm eşleşmeleri değiştir
sed 's/old/new/g' file.txt

# Case-insensitive
sed 's/old/new/gi' file.txt

# Dosyada değiştir (in-place)
sed -i 's/old/new/g' file.txt

# Backup al
sed -i.bak 's/old/new/g' file.txt
```

---

### Satır İşlemleri

```bash
# Belirli satırı yazdır
sed -n '5p' file.txt

# Satır aralığı
sed -n '5,10p' file.txt

# Son satır
sed -n '$p' file.txt

# Satır sil
sed '5d' file.txt

# Satır aralığı sil
sed '5,10d' file.txt

# Pattern eşleşen satır sil
sed '/pattern/d' file.txt

# Boş satır sil
sed '/^$/d' file.txt

# Yorum satırları sil
sed '/^#/d' file.txt
```

---

### Satır Ekleme

```bash
# Satır sonuna ekle (append)
sed 'a\new line' file.txt

# Satır başına ekle (insert)
sed 'i\new line' file.txt

# Belirli satırdan sonra ekle
sed '5a\new line' file.txt

# Pattern'den sonra ekle
sed '/pattern/a\new line' file.txt
```

---

### Gelişmiş Kullanım

```bash
# Birden fazla komut
sed -e 's/old/new/g' -e 's/foo/bar/g' file.txt

# Script dosyasından oku
sed -f script.sed file.txt

# Belli satırlarda değiştir
sed '5,10s/old/new/g' file.txt

# Pattern eşleşen satırda değiştir
sed '/error/s/red/blue/g' file.txt

# N. eşleşmeyi değiştir (2. eşleşme)
sed 's/pattern/replace/2' file.txt

# Regex grubu kullan
sed 's/\([0-9]*\)-\([0-9]*\)/\2-\1/' file.txt

# Delimiter değiştir (/ yerine |)
sed 's|/old/path|/new/path|g' file.txt
```

---

### Pratik sed Örnekler

```bash
# IP değiştir
sed -i 's/192.168.1.100/192.168.2.100/g' config.conf

# Port değiştir
sed -i 's/^Port 22/Port 2222/' /etc/ssh/sshd_config

# Yorum satırı ekle
sed -i '1i\# Config file' config.conf

# Tab'ları space'e çevir
sed 's/\t/    /g' file.txt

# Satır sonundaki boşlukları sil
sed 's/[[:space:]]*$//' file.txt

# Windows satır sonlarını temizle
sed 's/\r$//' file.txt

# Email'leri gizle
sed 's/[a-zA-Z0-9._%+-]*@[a-zA-Z0-9.-]*\.[a-zA-Z]*/***@***.***/' file.txt
```

---

## awk - Text Processing Language

### Basit Kullanım

```bash
# Tüm satırları yazdır
awk '{print}' file.txt

# İlk sütun
awk '{print $1}' file.txt

# Birden fazla sütun
awk '{print $1, $3}' file.txt

# Delimiter belirt
awk -F: '{print $1}' /etc/passwd

# Custom format
awk '{print "User:", $1, "Shell:", $7}' /etc/passwd
```

---

### Built-in Variables

```bash
$0    # Tüm satır
$1, $2, $3...  # Sütunlar
NF    # Sütun sayısı
NR    # Satır numarası
FS    # Field separator (delimiter)
OFS   # Output field separator
RS    # Record separator
ORS   # Output record separator
```

**Örnekler:**
```bash
# Satır numarası ile
awk '{print NR, $0}' file.txt

# Son sütun
awk '{print $NF}' file.txt

# Sütun sayısı
awk '{print NF}' file.txt

# Output delimiter
awk -F: -v OFS=',' '{print $1, $3}' /etc/passwd
```

---

### Pattern Matching

```bash
# Pattern içeren satırlar
awk '/pattern/ {print}' file.txt

# Pattern ile başlayan
awk '/^root/ {print}' /etc/passwd

# Pattern içermeyen
awk '!/pattern/ {print}' file.txt

# Belirli sütunda pattern
awk '$3 > 1000 {print $1}' /etc/passwd
```

---

### Koşullar

```bash
# Sayısal karşılaştırma
awk '$3 > 1000 {print $1}' /etc/passwd

# String karşılaştırma
awk '$7 == "/bin/bash" {print $1}' /etc/passwd

# Multiple conditions
awk '$3 > 1000 && $7 == "/bin/bash" {print $1}' /etc/passwd

# if-else
awk '{if ($3 > 1000) print $1 " (regular)"; else print $1 " (system)"}' /etc/passwd
```

---

### BEGIN ve END

```bash
# BEGIN: İşlem başında çalışır
awk 'BEGIN {print "User List:"} {print $1}' /etc/passwd

# END: İşlem sonunda çalışır
awk '{sum += $1} END {print "Total:", sum}' numbers.txt

# Hem BEGIN hem END
awk 'BEGIN {print "START"} {print} END {print "END"}' file.txt
```

---

### Matematiksel İşlemler

```bash
# Toplam
awk '{sum += $1} END {print sum}' numbers.txt

# Ortalama
awk '{sum += $1} END {print sum/NR}' numbers.txt

# Maksimum
awk 'BEGIN {max = 0} {if ($1 > max) max = $1} END {print max}' numbers.txt

# Sayım
awk 'END {print NR}' file.txt
```

---

### Pratik awk Örnekleri

```bash
# Normal kullanıcıları listele (UID >= 1000)
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# Disk kullanımı (sadece yüzde)
df -h | awk 'NR > 1 {print $5}'

# Memory kullanımı
free -h | awk 'NR == 2 {print $3}'

# En çok CPU kullanan 5 process
ps aux | awk 'NR > 1 {print $3, $11}' | sort -rn | head -5

# Log'da IP'leri say
awk '{print $1}' access.log | sort | uniq -c | sort -rn

# CSV parse
awk -F, '{print $1, $3}' data.csv

# JSON değer çıkar (basit)
awk -F'"' '/key/ {print $4}' data.json

# Belirli tarih aralığı (log)
awk '/2024-01-15/,/2024-01-16/ {print}' log.txt
```

---

### Gelişmiş awk

```bash
# Fonksiyon tanımla
awk 'function abs(x) {return x < 0 ? -x : x} {print abs($1)}' file.txt

# Array kullan
awk '{arr[$1]++} END {for (i in arr) print i, arr[i]}' file.txt

# Loop
awk 'BEGIN {for (i=1; i<=10; i++) print i}'

# String fonksiyonları
awk '{print toupper($1)}' file.txt     # UPPERCASE
awk '{print tolower($1)}' file.txt     # lowercase
awk '{print length($1)}' file.txt      # Uzunluk
awk '{print substr($1, 1, 3)}' file.txt  # Substring
```

---

## cut - Sütun Kesme

```bash
# Karakter bazlı
cut -c 1-5 file.txt

# Delimiter bazlı
cut -d: -f1 /etc/passwd

# Birden fazla sütun
cut -d: -f1,3,7 /etc/passwd

# Sütun aralığı
cut -d: -f1-3 /etc/passwd

# Tab delimiter
cut -f1,2 file.tsv

# Space delimiter
cut -d' ' -f1 file.txt
```

---

## sort - Sıralama

```bash
# Alfabetik sıralama
sort file.txt

# Ters sıralama
sort -r file.txt

# Sayısal sıralama
sort -n numbers.txt

# Unique (tekrarsız)
sort -u file.txt

# Belirli sütuna göre
sort -k2 file.txt

# Delimiter belirt
sort -t: -k3 -n /etc/passwd

# Human-readable sayılarla (1K, 2M, 3G)
du -h | sort -h

# Random sıralama
sort -R file.txt
```

---

## uniq - Tekrar Eleme

```bash
# Ardışık tekrarları sil
uniq file.txt

# Tekrar sayısını göster
uniq -c file.txt

# Sadece tekrar edenleri göster
uniq -d file.txt

# Sadece unique olanları göster
uniq -u file.txt

# Case-insensitive
uniq -i file.txt

# sort ile kombine (zorunlu)
sort file.txt | uniq
sort file.txt | uniq -c | sort -rn  # En çok tekrar eden
```

---

## tr - Karakter Değiştirme

```bash
# Küçük harfe çevir
echo "HELLO" | tr 'A-Z' 'a-z'

# Büyük harfe çevir
echo "hello" | tr 'a-z' 'A-Z'

# Karakter sil
echo "hello123" | tr -d '0-9'

# Boşlukları sil
echo "hello world" | tr -d ' '

# Boşlukları newline yap
echo "hello world foo" | tr ' ' '\n'

# Squeeze (ardışık tekrarları tek yap)
echo "hello    world" | tr -s ' '

# ROT13 encoding
echo "hello" | tr 'a-zA-Z' 'n-za-mN-ZA-M'
```

---

## wc - Word Count

```bash
# Satır, kelime, byte
wc file.txt

# Sadece satır sayısı
wc -l file.txt

# Sadece kelime sayısı
wc -w file.txt

# Sadece karakter sayısı
wc -m file.txt

# Sadece byte
wc -c file.txt

# Birden fazla dosya
wc -l *.txt
```

---

## head ve tail

```bash
# İlk 10 satır
head file.txt

# İlk N satır
head -n 20 file.txt
head -20 file.txt

# Son 10 satır
tail file.txt

# Son N satır
tail -n 20 file.txt

# Canlı takip (log izleme)
tail -f /var/log/syslog

# Canlı takip (dosya silinse bile devam et)
tail -F /var/log/app.log

# İlk N satır hariç hepsi
tail -n +11 file.txt

# Son N satır hariç hepsi
head -n -10 file.txt
```

---

## diff - Dosya Karşılaştırma

```bash
# İki dosya karşılaştır
diff file1.txt file2.txt

# Unified format (patch için)
diff -u file1.txt file2.txt

# Side-by-side
diff -y file1.txt file2.txt

# Sadece farklı mı? (exit code)
diff -q file1.txt file2.txt

# Dizin karşılaştırma
diff -r dir1/ dir2/

# Context format
diff -c file1.txt file2.txt
```

---

## paste ve join

### paste - Dosyaları Yan Yana

```bash
# İki dosyayı yan yana yapıştır
paste file1.txt file2.txt

# Custom delimiter
paste -d ',' file1.txt file2.txt

# Serial (satır satır)
paste -s file.txt
```

---

### join - Ortak Alana Göre Birleştir

```bash
# İlk sütuna göre join
join file1.txt file2.txt

# Belirli alanlara göre
join -1 2 -2 1 file1.txt file2.txt
```

---

## Pratik Senaryolar

### 1. Log Analizi - En Çok Erişen IP'ler

```bash
# Nginx access log
awk '{print $1}' /var/log/nginx/access.log | \
  sort | uniq -c | sort -rn | head -10
```

---

### 2. Başarısız SSH Girişleri

```bash
sudo grep "Failed password" /var/log/auth.log | \
  awk '{print $(NF-3)}' | \
  sort | uniq -c | sort -rn
```

---

### 3. Disk Kullanımı - En Büyük 10 Dizin

```bash
du -h /home | sort -rh | head -10
```

---

### 4. Process Memory Kullanımı

```bash
ps aux | awk '{print $6, $11}' | sort -rn | head -10
```

---

### 5. CSV Parse ve İstatistik

```bash
# users.csv (name,age,city)
# Yaşların ortalaması
awk -F, 'NR > 1 {sum += $2; count++} END {print sum/count}' users.csv

# Şehir bazında sayım
awk -F, 'NR > 1 {cities[$3]++} END {for (city in cities) print city, cities[city]}' users.csv
```

---

### 6. Web Server Log - 404 Hataları

```bash
awk '$9 == 404 {print $7}' /var/log/nginx/access.log | \
  sort | uniq -c | sort -rn | head -20
```

---

### 7. Config Dosyasında Aktif Satırlar

```bash
# Yorum ve boş satırları çıkar
grep -v '^#' /etc/ssh/sshd_config | grep -v '^$'

# veya sed ile
sed '/^#/d; /^$/d' /etc/ssh/sshd_config
```

---

### 8. Email Adresleri Çıkarma

```bash
grep -Eoh '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b' file.txt | \
  sort -u
```

---

### 9. Log Tarihe Göre Filtreleme

```bash
# Belirli saat aralığı
awk '/15:00/,/16:00/ {print}' /var/log/app.log

# Belirli tarih
sed -n '/2024-01-15/,/2024-01-16/p' /var/log/app.log
```

---

### 10. JSON Parse (basit)

```bash
# {"name": "Yasin", "age": 25}
# Name çıkar
grep -oP '(?<="name": ")[^"]*' data.json

# veya awk
awk -F'"' '/"name"/ {print $4}' data.json
```

---

## Komut Zincirleme (Piping)

```bash
# Pipe ile komut zincirleme
cat file.txt | grep "error" | sort | uniq -c

# Daha verimli (cat gereksiz)
grep "error" file.txt | sort | uniq -c

# Çok adımlı analiz
ps aux | \
  awk '{print $3, $11}' | \
  sort -rn | \
  head -10 | \
  column -t

# tee ile hem ekrana hem dosyaya
grep "error" log.txt | tee errors.txt | wc -l
```

---

## 🎯 Özet

| Komut | Açıklama | Örnek |
|-------|----------|-------|
| `grep` | Pattern ara | `grep "error" log.txt` |
| `sed` | Stream editor | `sed 's/old/new/g' file.txt` |
| `awk` | Text processing | `awk '{print $1}' file.txt` |
| `cut` | Sütun kes | `cut -d: -f1 /etc/passwd` |
| `sort` | Sırala | `sort -n numbers.txt` |
| `uniq` | Tekrar eleme | `sort file | uniq -c` |
| `tr` | Karakter değiştir | `tr 'a-z' 'A-Z'` |
| `wc` | Sayım | `wc -l file.txt` |

---

## 🎓 Alıştırmalar

1. Log dosyasında "error" içeren satırları bul
2. /etc/passwd'da sadece kullanıcı adlarını listele
3. IP adreslerini text'ten çıkar (regex)
4. CSV dosyasında 3. sütunu yazdır
5. En çok tekrar eden kelimeleri bul
6. İki dosyanın farkını al
7. Log'da belirli tarih aralığındaki kayıtlar
8. Email adreslerini gizle (sed ile)
9. Process'leri CPU kullanımına göre sırala
10. JSON'dan belirli değer çıkar

---

**Sonraki Bölüm:** [10-Servis-Init-Systemd.md](10-Servis-Init-Systemd.md) → Systemd derinlemesine, unit dosyaları, service yönetimi
