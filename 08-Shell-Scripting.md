# 🐧 LINUX REHBERİ — BÖLÜM 8: SHELL SCRIPTING (BASH)

Bash scripting: otomasyon, kontrol yapıları, fonksiyonlar, best practices.

---

## Shell Nedir?

**Shell**: Kullanıcı ile kernel arasındaki arayüz.

### Shell Tipleri

- **sh**: Bourne Shell (eski)
- **bash**: Bourne Again Shell (en yaygın)
- **zsh**: Z Shell (modern, Oh My Zsh)
- **fish**: Friendly Interactive Shell
- **dash**: Debian Almquist Shell (hızlı)

```bash
# Hangi shell kullanıyorum?
echo $SHELL

# Mevcut shelleer
cat /etc/shells

# Shell değiştir
chsh -s /bin/zsh
```

---

## İlk Script

### Shebang (#!)

Script'in hangi interpreter ile çalışacağını belirtir.

```bash
#!/bin/bash
# İlk scriptim

echo "Merhaba Dünya!"
```

**Çalıştırma:**
```bash
# İzin ver
chmod +x script.sh

# Çalıştır
./script.sh

# Veya
bash script.sh
```

**Shebang alternatifleri:**
```bash
#!/bin/bash           # Bash
#!/bin/sh             # POSIX shell
#!/usr/bin/env bash   # Bash'i PATH'te ara (portable)
#!/usr/bin/python3    # Python
```

---

## Değişkenler

### Tanımlama ve Kullanım

```bash
#!/bin/bash

# Değişken tanımla (boşluk yok!)
NAME="Yasin"
AGE=25

# Kullan ($ ile)
echo "Merhaba $NAME"
echo "Yaşın: $AGE"

# Süslü parantez (opsiyonel ama önerilir)
echo "Merhaba ${NAME}"

# Komut çıktısını değişkene ata
CURRENT_DATE=$(date)
FILES=$(ls)

# Veya backticks (eski yöntem)
CURRENT_DATE=`date`
```

---

### Read-only Değişkenler

```bash
readonly PI=3.14159
PI=3.14  # Hata!

declare -r MAX_SIZE=1000
```

---

### Değişken Silme

```bash
NAME="Yasin"
unset NAME
echo $NAME  # Boş
```

---

## Özel Değişkenler

```bash
#!/bin/bash

echo "Script adı: $0"
echo "1. parametre: $1"
echo "2. parametre: $2"
echo "Tüm parametreler: $@"
echo "Parametre sayısı: $#"
echo "Son komutun exit kodu: $?"
echo "Script PID: $$"
echo "Son background process PID: $!"
```

**Kullanım:**
```bash
./script.sh arg1 arg2 arg3
# $0 = ./script.sh
# $1 = arg1
# $2 = arg2
# $@ = arg1 arg2 arg3
# $# = 3
```

---

## Ortam Değişkenleri

```bash
# Ortam değişkeni tanımla
export PATH="/usr/local/bin:$PATH"
export DB_HOST="localhost"

# Kullan
echo $PATH
echo $HOME
echo $USER
echo $PWD

# Tüm ortam değişkenleri
env
printenv

# Belirli değişken
printenv HOME
```

---

## String İşlemleri

### String Birleştirme

```bash
FIRST="Yasin"
LAST="Doe"
FULL="${FIRST} ${LAST}"
echo $FULL  # Yasin Doe
```

---

### String Uzunluğu

```bash
TEXT="Merhaba"
echo ${#TEXT}  # 7
```

---

### Substring

```bash
TEXT="Merhaba Dünya"
echo ${TEXT:0:7}   # Merhaba
echo ${TEXT:8}     # Dünya
```

---

### String Replace

```bash
TEXT="Merhaba Dünya"
echo ${TEXT/Dünya/Linux}  # Merhaba Linux

# Tüm oluşumları değiştir
TEXT="foo bar foo"
echo ${TEXT//foo/baz}  # baz bar baz
```

---

### String Büyük/Küçük Harf

```bash
TEXT="Merhaba"
echo ${TEXT^^}  # MERHABA (uppercase)
echo ${TEXT,,}  # merhaba (lowercase)
```

---

### String Trimming

```bash
TEXT="   merhaba   "
# Baştan boşluk sil
echo "${TEXT#"${TEXT%%[![:space:]]*}"}"

# Trim fonksiyonu
trim() {
    local var="$*"
    var="${var#"${var%%[![:space:]]*}"}"
    var="${var%"${var##*[![:space:]]}"}"
    echo "$var"
}

echo "$(trim "$TEXT")"
```

---

## Koşullu İfadeler (if/else)

### Basit if

```bash
#!/bin/bash

AGE=18

if [ $AGE -ge 18 ]; then
    echo "Reşitsiniz"
fi
```

---

### if-else

```bash
#!/bin/bash

AGE=16

if [ $AGE -ge 18 ]; then
    echo "Reşitsiniz"
else
    echo "Reşit değilsiniz"
fi
```

---

### if-elif-else

```bash
#!/bin/bash

SCORE=75

if [ $SCORE -ge 90 ]; then
    echo "A"
elif [ $SCORE -ge 80 ]; then
    echo "B"
elif [ $SCORE -ge 70 ]; then
    echo "C"
else
    echo "F"
fi
```

---

### Test Operatörleri

#### Sayısal Karşılaştırma

```bash
-eq  # Equal (eşit)
-ne  # Not equal (eşit değil)
-gt  # Greater than (büyük)
-ge  # Greater or equal (büyük veya eşit)
-lt  # Less than (küçük)
-le  # Less or equal (küçük veya eşit)
```

**Örnek:**
```bash
if [ $NUM -eq 10 ]; then
    echo "10"
fi
```

---

#### String Karşılaştırma

```bash
=    # Eşit
!=   # Eşit değil
-z   # Boş string
-n   # Boş değil
```

**Örnek:**
```bash
if [ "$NAME" = "Yasin" ]; then
    echo "Merhaba Yasin"
fi

if [ -z "$VAR" ]; then
    echo "VAR boş"
fi
```

---

#### Dosya Testleri

```bash
-e   # Dosya var mı?
-f   # Normal dosya mı?
-d   # Dizin mi?
-r   # Okunabilir mi?
-w   # Yazılabilir mi?
-x   # Çalıştırılabilir mi?
-s   # Dosya boş değil mi?
-L   # Symbolic link mi?
```

**Örnek:**
```bash
if [ -f "/etc/passwd" ]; then
    echo "Dosya var"
fi

if [ -d "/home/yasin" ]; then
    echo "Dizin var"
fi

if [ -x "script.sh" ]; then
    echo "Çalıştırılabilir"
fi
```

---

### Mantıksal Operatörler

```bash
&&   # VE (AND)
||   # VEYA (OR)
!    # DEĞİL (NOT)
```

**Örnek:**
```bash
if [ $AGE -ge 18 ] && [ $AGE -le 65 ]; then
    echo "Çalışma yaşında"
fi

if [ $AGE -lt 18 ] || [ $AGE -gt 65 ]; then
    echo "Çalışma yaşında değil"
fi

if [ ! -f "file.txt" ]; then
    echo "Dosya yok"
fi
```

---

### Modern Test Syntax: [[  ]]

```bash
# String pattern matching
if [[ $NAME == Ya* ]]; then
    echo "İsim Ya ile başlıyor"
fi

# Regex
if [[ $EMAIL =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Geçerli email"
fi

# Mantıksal operatörler
if [[ $AGE -ge 18 && $AGE -le 65 ]]; then
    echo "Çalışma yaşında"
fi
```

**[ ] vs [[ ]]:**
- `[[ ]]`: Bash özgü, daha güçlü
- `[ ]`: POSIX uyumlu, portable

---

## case Yapısı

Switch-case gibi.

```bash
#!/bin/bash

read -p "Bir sayı gir (1-3): " NUM

case $NUM in
    1)
        echo "Bir"
        ;;
    2)
        echo "İki"
        ;;
    3)
        echo "Üç"
        ;;
    *)
        echo "Geçersiz"
        ;;
esac
```

**Wildcard kullanımı:**
```bash
#!/bin/bash

read -p "Dosya adı: " FILE

case $FILE in
    *.txt)
        echo "Text dosyası"
        ;;
    *.jpg|*.png)
        echo "Resim dosyası"
        ;;
    *.sh)
        echo "Shell script"
        ;;
    *)
        echo "Bilinmeyen tip"
        ;;
esac
```

---

## Döngüler

### for Döngüsü

#### Liste üzerinde

```bash
#!/bin/bash

# String listesi
for NAME in Yasin Ali Veli Ayşe
do
    echo "Merhaba $NAME"
done

# Dosyalar
for FILE in *.txt
do
    echo "Dosya: $FILE"
done

# Command output
for USER in $(cat /etc/passwd | cut -d: -f1)
do
    echo "Kullanıcı: $USER"
done
```

---

#### C-style for

```bash
#!/bin/bash

for ((i=1; i<=10; i++))
do
    echo "Sayı: $i"
done
```

---

#### Seq ile

```bash
#!/bin/bash

for i in $(seq 1 10)
do
    echo "Sayı: $i"
done

# Veya brace expansion
for i in {1..10}
do
    echo "Sayı: $i"
done

# Step ile
for i in {0..100..10}
do
    echo "Sayı: $i"
done
```

---

### while Döngüsü

```bash
#!/bin/bash

COUNT=1

while [ $COUNT -le 5 ]
do
    echo "Count: $COUNT"
    ((COUNT++))
done
```

**Dosya okuma:**
```bash
#!/bin/bash

while read LINE
do
    echo "Satır: $LINE"
done < file.txt
```

---

### until Döngüsü

```bash
#!/bin/bash

COUNT=1

until [ $COUNT -gt 5 ]
do
    echo "Count: $COUNT"
    ((COUNT++))
done
```

---

### break ve continue

```bash
#!/bin/bash

# break: Döngüden çık
for i in {1..10}
do
    if [ $i -eq 5 ]; then
        break
    fi
    echo $i
done

# continue: Sonraki iterasyona geç
for i in {1..10}
do
    if [ $i -eq 5 ]; then
        continue
    fi
    echo $i
done
```

---

## Fonksiyonlar

### Basit Fonksiyon

```bash
#!/bin/bash

# Fonksiyon tanımla
greet() {
    echo "Merhaba!"
}

# Çağır
greet
```

---

### Parametreli Fonksiyon

```bash
#!/bin/bash

greet() {
    local NAME=$1
    echo "Merhaba $NAME!"
}

greet "Yasin"
greet "Ali"
```

---

### Return Value

```bash
#!/bin/bash

add() {
    local RESULT=$(($1 + $2))
    echo $RESULT
}

TOTAL=$(add 5 3)
echo "Toplam: $TOTAL"
```

---

### Return Status

```bash
#!/bin/bash

is_root() {
    if [ $EUID -eq 0 ]; then
        return 0  # Success
    else
        return 1  # Failure
    fi
}

if is_root; then
    echo "Root kullanıcısınız"
else
    echo "Root değilsiniz"
fi
```

---

### Local vs Global Değişken

```bash
#!/bin/bash

GLOBAL="Global değer"

test_func() {
    local LOCAL="Local değer"
    GLOBAL="Değişti"
    echo $LOCAL
    echo $GLOBAL
}

test_func
echo $GLOBAL  # Değişti
echo $LOCAL   # Boş (local scope)
```

---

## Kullanıcı Input

### read Komutu

```bash
#!/bin/bash

# Basit input
read -p "Adınız: " NAME
echo "Merhaba $NAME"

# Şifre (görünmez)
read -sp "Şifre: " PASSWORD
echo
echo "Şifre alındı"

# Timeout
read -t 5 -p "5 saniyede cevap verin: " ANSWER

# Default değer
read -p "İsim (default: Yasin): " NAME
NAME=${NAME:-Yasin}
echo "İsim: $NAME"
```

---

### select Menü

```bash
#!/bin/bash

select OPTION in "Seçenek 1" "Seçenek 2" "Seçenek 3" "Çıkış"
do
    case $OPTION in
        "Seçenek 1")
            echo "1 seçildi"
            ;;
        "Seçenek 2")
            echo "2 seçildi"
            ;;
        "Seçenek 3")
            echo "3 seçildi"
            ;;
        "Çıkış")
            break
            ;;
        *)
            echo "Geçersiz"
            ;;
    esac
done
```

---

## Arrays (Diziler)

### Array Tanımlama

```bash
#!/bin/bash

# Array oluştur
NAMES=("Yasin" "Ali" "Veli")

# Eleman ekle
NAMES+=("Ayşe")

# Elemana eriş
echo ${NAMES[0]}  # Yasin
echo ${NAMES[1]}  # Ali

# Tüm elemanlar
echo ${NAMES[@]}

# Array uzunluğu
echo ${#NAMES[@]}

# Belirli index'ten itibaren
echo ${NAMES[@]:1:2}  # Ali Veli
```

---

### Array Döngüsü

```bash
#!/bin/bash

FRUITS=("Elma" "Armut" "Muz")

# Tüm elemanlar
for FRUIT in "${FRUITS[@]}"
do
    echo $FRUIT
done

# Index ile
for i in "${!FRUITS[@]}"
do
    echo "$i: ${FRUITS[$i]}"
done
```

---

## Aritmetik İşlemler

### let Komutu

```bash
#!/bin/bash

let "a = 5 + 3"
echo $a  # 8

let "a++"
echo $a  # 9

let "a *= 2"
echo $a  # 18
```

---

### $(( )) Syntax

```bash
#!/bin/bash

a=$((5 + 3))
echo $a  # 8

b=$((a * 2))
echo $b  # 16

((c = a + b))
echo $c  # 24
```

---

### expr Komutu (eski)

```bash
#!/bin/bash

a=5
b=3
SUM=$(expr $a + $b)
echo $SUM  # 8
```

---

### bc ile Ondalıklı

```bash
#!/bin/bash

# Ondalıklı işlem
RESULT=$(echo "scale=2; 10 / 3" | bc)
echo $RESULT  # 3.33

# Pi hesaplama
PI=$(echo "scale=10; 4*a(1)" | bc -l)
echo $PI
```

---

## Hata Yönetimi

### Exit Status

```bash
#!/bin/bash

# Exit kodunu kontrol et
ls /nonexistent
if [ $? -ne 0 ]; then
    echo "Komut başarısız"
fi

# Direkt kontrol
if ls /nonexistent; then
    echo "Başarılı"
else
    echo "Başarısız"
fi
```

---

### set -e (Hata durumunda dur)

```bash
#!/bin/bash
set -e  # İlk hatada dur

echo "1. komut"
false  # Bu başarısız olur
echo "Bu çalışmaz"
```

---

### trap Komutu

Signal yakalama.

```bash
#!/bin/bash

# Cleanup fonksiyonu
cleanup() {
    echo "Temizlik yapılıyor..."
    rm -f /tmp/tempfile
}

# EXIT sinyalinde cleanup çağır
trap cleanup EXIT

# Script devam eder...
echo "Script çalışıyor..."
touch /tmp/tempfile
sleep 5

# Ctrl+C yakalanması
trap "echo 'Interrupted!'; exit" SIGINT SIGTERM
```

---

## Debugging

### set -x (Debug mode)

```bash
#!/bin/bash
set -x  # Her komutu yazdır

NAME="Yasin"
echo "Merhaba $NAME"

set +x  # Debug mode kapat
```

---

### Kısmi Debug

```bash
#!/bin/bash

# Debug ettirmek istediğin kısım
set -x
for i in {1..3}
do
    echo $i
done
set +x

# Normal devam
echo "Bitti"
```

---

## Pratik Script Örnekleri

### 1. Backup Script

```bash
#!/bin/bash

# Değişkenler
SOURCE="/home/yasin"
DEST="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"

# Root kontrolü
if [ $EUID -ne 0 ]; then
    echo "Root olarak çalıştırın!"
    exit 1
fi

# Backup dizini var mı?
if [ ! -d "$DEST" ]; then
    mkdir -p "$DEST"
fi

# Backup al
echo "Backup alınıyor: $SOURCE -> $DEST/$BACKUP_FILE"
tar -czf "$DEST/$BACKUP_FILE" "$SOURCE" 2>/dev/null

# Kontrol
if [ $? -eq 0 ]; then
    echo "Backup başarılı!"
    ls -lh "$DEST/$BACKUP_FILE"
else
    echo "Backup başarısız!"
    exit 1
fi

# Eski backupları sil (30 günden eski)
find "$DEST" -name "backup_*.tar.gz" -mtime +30 -delete
echo "Eski backuplar temizlendi."
```

---

### 2. System Info Script

```bash
#!/bin/bash

echo "===== SİSTEM BİLGİLERİ ====="
echo

echo "Hostname: $(hostname)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"
echo

echo "CPU:"
lscpu | grep "Model name" | cut -d ':' -f2 | xargs
echo "Cores: $(nproc)"
echo

echo "Memory:"
free -h | grep Mem | awk '{print "Total: "$2" | Used: "$3" | Free: "$4}'
echo

echo "Disk:"
df -h / | tail -1 | awk '{print "Total: "$2" | Used: "$3" | Free: "$4" | Use%: "$5}'
echo

echo "Network:"
ip -br addr show | grep UP
echo

echo "Top 5 CPU Processes:"
ps aux --sort=-%cpu | head -6
echo

echo "Top 5 Memory Processes:"
ps aux --sort=-%mem | head -6
```

---

### 3. User Management Script

```bash
#!/bin/bash

# Fonksiyonlar
add_user() {
    read -p "Kullanıcı adı: " USERNAME
    read -sp "Şifre: " PASSWORD
    echo
    
    sudo useradd -m -s /bin/bash "$USERNAME"
    echo "$USERNAME:$PASSWORD" | sudo chpasswd
    echo "Kullanıcı eklendi: $USERNAME"
}

delete_user() {
    read -p "Silinecek kullanıcı: " USERNAME
    sudo userdel -r "$USERNAME"
    echo "Kullanıcı silindi: $USERNAME"
}

list_users() {
    echo "Normal kullanıcılar (UID >= 1000):"
    awk -F: '$3 >= 1000 {print $1" (UID: "$3")"}' /etc/passwd
}

# Menü
while true
do
    echo
    echo "===== KULLANICI YÖNETİMİ ====="
    echo "1. Kullanıcı Ekle"
    echo "2. Kullanıcı Sil"
    echo "3. Kullanıcıları Listele"
    echo "4. Çıkış"
    read -p "Seçiminiz: " CHOICE
    
    case $CHOICE in
        1) add_user ;;
        2) delete_user ;;
        3) list_users ;;
        4) echo "Çıkılıyor..."; exit 0 ;;
        *) echo "Geçersiz seçim!" ;;
    esac
done
```

---

### 4. Service Monitor Script

```bash
#!/bin/bash

# İzlenecek servisler
SERVICES=("nginx" "mysql" "redis")

# Email (opsiyonel)
EMAIL="admin@example.com"

for SERVICE in "${SERVICES[@]}"
do
    if systemctl is-active --quiet "$SERVICE"; then
        echo "✓ $SERVICE çalışıyor"
    else
        echo "✗ $SERVICE çalışmıyor!"
        
        # Servisi başlat
        sudo systemctl start "$SERVICE"
        
        # Email gönder (mail komutu kurulu olmalı)
        # echo "$SERVICE servisi düştü ve yeniden başlatıldı" | mail -s "Service Alert" "$EMAIL"
    fi
done
```

---

## Best Practices

### ✅ Yapılması Gerekenler

1. **Shebang kullan**
   ```bash
   #!/bin/bash
   ```

2. **Hata kontrolü**
   ```bash
   set -e  # Hatalarda dur
   set -u  # Tanımsız değişkenlerde dur
   set -o pipefail  # Pipe zinciride hata varsa dur
   ```

3. **Değişkenleri quote et**
   ```bash
   echo "$VAR"  # Boşluk içerebilir
   ```

4. **Local değişken kullan**
   ```bash
   my_func() {
       local VAR="value"
   }
   ```

5. **Exit kod kontrol et**
   ```bash
   if command; then
       echo "Başarılı"
   fi
   ```

---

### ❌ Yapılmaması Gerekenler

1. ❌ **Quote kullanmama**
   ```bash
   # YANLIŞ
   rm $FILE
   
   # DOĞRU
   rm "$FILE"
   ```

2. ❌ **Boşluk bırakma (atama)**
   ```bash
   # YANLIŞ
   VAR = "value"
   
   # DOĞRU
   VAR="value"
   ```

3. ❌ **[ ] yerine == kullanma**
   ```bash
   # YANLIŞ
   if [ $VAR == "value" ]
   
   # DOĞRU
   if [ "$VAR" = "value" ]
   ```

---

## 🎯 Özet

| Konu | Syntax | Örnek |
|------|--------|-------|
| Değişken | `VAR=value` | `NAME="Yasin"` |
| Koşul | `if [ ]; then fi` | `if [ $A -gt 5 ]; then echo "OK"; fi` |
| Döngü | `for i in list; do done` | `for i in {1..10}; do echo $i; done` |
| Fonksiyon | `func() { }` | `greet() { echo "Hi $1"; }` |
| Array | `ARR=(a b c)` | `echo ${ARR[0]}` |
| String | `${VAR}` | `echo "Hello ${NAME}"` |

---

## 🎓 Alıştırmalar

1. "Merhaba Dünya" yazdıran basit script
2. Kullanıcıdan isim al ve selamla
3. 1'den 100'e kadar çift sayılar
4. Dosya var mı kontrol et
5. Toplama yapan fonksiyon
6. Array'deki tüm elemanları yazdır
7. Case ile menü yap
8. Backup script (tar ile)
9. System monitoring script
10. Kullanıcı yönetim scripti

---

**Sonraki Bölüm:** [09-Text-Isleme.md](09-Text-Isleme.md) → grep, sed, awk ve text processing
