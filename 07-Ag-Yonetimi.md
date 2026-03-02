# 🐧 LINUX REHBERİ — BÖLÜM 7: AĞ YÖNETİMİ

Linux networking: IP yapılandırma, DNS, routing, firewall, troubleshooting.

---

## Network Interface Yönetimi

### ip komutu (modern)

`ifconfig` yerine artık `ip` kullanılır.

```bash
# Tüm interface'leri göster
ip addr
ip a

# Belirli interface
ip addr show eth0

# Interface'leri kısa listele
ip -br addr

# Link durumu
ip link
ip link show eth0

# Interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down
```

---

### IP Adresi Ekleme/Silme

```bash
# IP ekle
sudo ip addr add 192.168.1.100/24 dev eth0

# IP sil
sudo ip addr del 192.168.1.100/24 dev eth0

# İkincil IP ekle
sudo ip addr add 192.168.1.101/24 dev eth0
```

---

### ifconfig (klasik)

```bash
# Tüm interface'ler
ifconfig

# Belirli interface
ifconfig eth0

# IP ata
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# Interface up/down
sudo ifconfig eth0 up
sudo ifconfig eth0 down
```

---

## Network Ayarları

### netplan (Ubuntu 18.04+)

```bash
# Config dosyası
sudo nano /etc/netplan/00-installer-config.yaml

# Örnek: Statik IP
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4

# Uygula
sudo netplan apply

# Test et (120 sn sonra geri alır)
sudo netplan try
```

**Örnek: DHCP:**
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      dhcp6: false
```

---

### NetworkManager (Desktop & RHEL/Rocky)

```bash
# nmcli ile interface listesi
nmcli device status

# Bağlantı listesi
nmcli connection show

# Statik IP ayarla
sudo nmcli connection modify eth0 ipv4.addresses 192.168.1.100/24
sudo nmcli connection modify eth0 ipv4.gateway 192.168.1.1
sudo nmcli connection modify eth0 ipv4.dns "8.8.8.8 8.8.4.4"
sudo nmcli connection modify eth0 ipv4.method manual

# DHCP'ye geçir
sudo nmcli connection modify eth0 ipv4.method auto

# Interface restart
sudo nmcli connection down eth0
sudo nmcli connection up eth0

# Yeni bağlantı oluştur
sudo nmcli connection add type ethernet con-name my-eth0 ifname eth0
```

---

### systemd-networkd

```bash
# Config dosyası
sudo nano /etc/systemd/network/20-wired.network

[Match]
Name=eth0

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
DNS=8.8.8.8
DNS=8.8.4.4

# Restart
sudo systemctl restart systemd-networkd
```

---

## Routing

### ip route

```bash
# Routing table
ip route
ip route show

# Default gateway
ip route show default

# Route ekle
sudo ip route add 10.0.0.0/24 via 192.168.1.1 dev eth0

# Route sil
sudo ip route del 10.0.0.0/24

# Default gateway değiştir
sudo ip route add default via 192.168.1.1
sudo ip route del default via 192.168.1.254
```

---

### Kalıcı Route (Ubuntu)

```bash
# Netplan ile
sudo nano /etc/netplan/00-installer-config.yaml

network:
  ethernets:
    eth0:
      routes:
        - to: 10.0.0.0/24
          via: 192.168.1.1
        - to: default
          via: 192.168.1.1
```

---

## DNS Yapılandırması

### /etc/resolv.conf

```bash
# DNS sunucuları
cat /etc/resolv.conf

nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com

# Manuel düzenle (geçici)
sudo nano /etc/resolv.conf
```

**⚠️ Ubuntu'da systemd-resolved yönetir, direkt düzenleme kaybolur!**

---

### systemd-resolved (Ubuntu)

```bash
# DNS durumu
resolvectl status

# DNS query
resolvectl query google.com

# DNS cache temizle
sudo resolvectl flush-caches

# Config
sudo nano /etc/systemd/resolved.conf

[Resolve]
DNS=8.8.8.8 8.8.4.4
FallbackDNS=1.1.1.1
Domains=example.com

# Restart
sudo systemctl restart systemd-resolved
```

---

### /etc/hosts

Statik DNS kayıtları.

```bash
# Düzenle
sudo nano /etc/hosts

# Format
# IP        hostname    aliases
127.0.0.1   localhost
192.168.1.10  server1.local  server1
10.0.0.50   database.local db
```

---

## Network Bağlantılar

### ping

ICMP echo request.

```bash
# Ping
ping google.com

# 4 paket gönder
ping -c 4 google.com

# Interval değiştir
ping -i 0.5 google.com

# Flood ping (root)
sudo ping -f google.com

# IPv6
ping6 google.com
```

---

### traceroute / tracepath

Yolu izle.

```bash
# Kur
sudo apt install traceroute

# Traceroute
traceroute google.com

# Tracepath (root gerektirmez)
tracepath google.com

# ICMP yerine TCP kullan
sudo traceroute -T google.com
```

---

### mtr

Kombine ping + traceroute.

```bash
# Kur
sudo apt install mtr

# İnteraktif
mtr google.com

# Report mode
mtr -r -c 10 google.com
```

---

## Port ve Soket Monitoring

### ss (modern)

`netstat` yerine `ss`.

```bash
# Tüm soketler
ss -a

# Listening portlar
ss -l

# TCP bağlantılar
ss -t

# UDP bağlantılar
ss -u

# Listening TCP portlar
ss -lt

# Process bilgisi ile
sudo ss -ltp

# Belirli port
ss -ltn sport = :80

# Established bağlantılar
ss -t state established

# Özet istatistik
ss -s
```

---

### netstat (klasik)

```bash
# Listening portlar
sudo netstat -tulpn
# -t: TCP
# -u: UDP
# -l: Listening
# -p: Process
# -n: Numeric (isim çözme)

# Tüm bağlantılar
netstat -an

# Routing table
netstat -r

# Interface istatistikleri
netstat -i
```

---

### lsof

Açık dosyalar ve portlar.

```bash
# Belirli port
sudo lsof -i :80

# Tüm network connections
sudo lsof -i

# TCP bağlantılar
sudo lsof -i tcp

# Belirli process
sudo lsof -p 1234

# Belirli kullanıcı
sudo lsof -u yasin
```

---

## Firewall Yönetimi

### ufw (Ubuntu - Uncomplicated Firewall)

Basit firewall yönetimi.

```bash
# Durumu kontrol et
sudo ufw status

# Aktif et
sudo ufw enable

# Deaktif et
sudo ufw disable

# Default policy
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Port aç
sudo ufw allow 22
sudo ufw allow 80/tcp
sudo ufw allow 443

# Port kapat
sudo ufw deny 23

# Belirli IP'ye izin ver
sudo ufw allow from 192.168.1.100

# IP ve port
sudo ufw allow from 192.168.1.100 to any port 22

# Subnet
sudo ufw allow from 192.168.1.0/24

# Kural sil
sudo ufw delete allow 80

# Numbered delete
sudo ufw status numbered
sudo ufw delete 2

# Reset (tüm kurallar sil)
sudo ufw reset

# Logging
sudo ufw logging on
sudo ufw logging high
```

---

### firewalld (RHEL/Rocky/Fedora)

Zone-based firewall.

```bash
# Durumu kontrol et
sudo firewall-cmd --state

# Aktif zonlar
sudo firewall-cmd --get-active-zones

# Default zone
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --set-default-zone=public

# Zone'daki servisler
sudo firewall-cmd --zone=public --list-all

# Servis ekle (geçici)
sudo firewall-cmd --zone=public --add-service=http

# Servis ekle (kalıcı)
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload

# Port ekle
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent

# IP'ye izin ver
sudo firewall-cmd --zone=public --add-source=192.168.1.100 --permanent

# Rich rule (gelişmiş)
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port port="22" protocol="tcp" accept' --permanent

# Kural sil
sudo firewall-cmd --zone=public --remove-service=http --permanent

# Reload
sudo firewall-cmd --reload
```

**Zones:**
- `drop`: Tüm bağlantılar red
- `block`: Reject ile red
- `public`: Genel network (default)
- `dmz`: DMZ sunucuları
- `work`: İş networkü
- `home`: Ev networkü
- `internal`: İç network
- `trusted`: Tümüne güven

---

### iptables (Low-level)

Kernel-level firewall.

```bash
# Kuralları listele
sudo iptables -L -v -n

# Tüm trafiğe izin ver (tehlikeli!)
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# Tüm gelen trafiği engelle
sudo iptables -P INPUT DROP

# Port 22'ye izin ver
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Belirli IP'ye izin ver
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Loopback'e izin ver
sudo iptables -A INPUT -i lo -j ACCEPT

# Established/related bağlantılara izin ver
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Kural sil
sudo iptables -D INPUT 3

# Chain temizle
sudo iptables -F

# Kuralları kaydet (Ubuntu)
sudo iptables-save > /etc/iptables/rules.v4

# Kuralları yükle
sudo iptables-restore < /etc/iptables/rules.v4
```

---

## Network Diagnostics

### nmap

Port scanning ve network discovery.

```bash
# Kur
sudo apt install nmap

# Basit scan
nmap 192.168.1.1

# Port range
nmap -p 1-1000 192.168.1.1

# Servis versiyonları
sudo nmap -sV 192.168.1.1

# OS detection
sudo nmap -O 192.168.1.1

# Subnet scan
nmap 192.168.1.0/24

# Hızlı scan (top 100 port)
nmap -F 192.168.1.1

# Tüm portlar
nmap -p- 192.168.1.1
```

---

### tcpdump

Packet capture.

```bash
# Tüm trafiği yakala
sudo tcpdump

# Belirli interface
sudo tcpdump -i eth0

# Belirli port
sudo tcpdump port 80

# Belirli host
sudo tcpdump host 192.168.1.100

# Dosyaya kaydet
sudo tcpdump -i eth0 -w capture.pcap

# Dosyadan oku
sudo tcpdump -r capture.pcap

# HTTP trafiği
sudo tcpdump -i eth0 'tcp port 80'

# DNS sorguları
sudo tcpdump -i eth0 port 53
```

---

### netcat (nc)

Swiss army knife.

```bash
# Port dinle
nc -l 1234

# Bağlan
nc 192.168.1.100 1234

# Port test
nc -zv google.com 80

# Dosya gönder
# Alıcı
nc -l 1234 > file.txt
# Gönderici
nc 192.168.1.100 1234 < file.txt

# Port scanning
nc -zv 192.168.1.1 20-80
```

---

### curl & wget

HTTP istekleri.

```bash
# wget ile download
wget https://example.com/file.zip

# Devam ettir
wget -c https://example.com/large.iso

# Recursive download (web sitesi)
wget -r -np -k https://example.com

# curl ile GET
curl https://api.example.com/data

# POST request
curl -X POST -d "key=value" https://api.example.com

# Headers
curl -H "Authorization: Bearer token" https://api.example.com

# Çıktıyı kaydet
curl -o output.html https://example.com

# Progress bar
curl -# -O https://example.com/file.zip

# Follow redirects
curl -L https://example.com
```

---

## WiFi Yönetimi

### iwconfig

Wireless interface.

```bash
# WiFi interface'leri göster
iwconfig

# SSID'ye bağlan
sudo iwconfig wlan0 essid "MyNetwork"
```

---

### nmcli (WiFi)

```bash
# WiFi durumu
nmcli radio wifi

# WiFi aç/kapat
nmcli radio wifi on
nmcli radio wifi off

# Mevcut networkleri tara
nmcli device wifi list

# Bağlan
nmcli device wifi connect "MySSID" password "mypassword"

# Bağlantıyı kes
nmcli device disconnect wlan0
```

---

## Bandwidth Monitoring

### iftop

Gerçek zamanlı bandwidth.

```bash
# Kur
sudo apt install iftop

# Çalıştır
sudo iftop

# Belirli interface
sudo iftop -i eth0
```

---

### nload

Basit bandwidth gösterge.

```bash
# Kur
sudo apt install nload

# Çalıştır
nload

# Belirli interface
nload eth0
```

---

### vnstat

Bandwidth istatistikleri.

```bash
# Kur ve başlat
sudo apt install vnstat
sudo systemctl enable --now vnstat

# Genel istatistik
vnstat

# Saatlik
vnstat -h

# Günlük
vnstat -d

# Aylık
vnstat -m

# Canlı monitoring
vnstat -l
```

---

## Bridge ve Bonding

### Bridge (Network köprüsü)

```bash
# Bridge oluştur
sudo ip link add name br0 type bridge

# Interface'leri ekle
sudo ip link set eth0 master br0
sudo ip link set eth1 master br0

# Bridge'i aktif et
sudo ip link set br0 up

# Kalıcı (netplan)
network:
  bridges:
    br0:
      interfaces:
        - eth0
        - eth1
      dhcp4: true
```

---

### Bonding (NIC teaming)

```bash
# Bonding modülü yükle
sudo modprobe bonding

# Bond oluştur
sudo ip link add bond0 type bond mode 802.3ad

# Interface'leri ekle
sudo ip link set eth0 master bond0
sudo ip link set eth1 master bond0
```

**Bonding modları:**
- `0` (balance-rr): Round-robin
- `1` (active-backup): Failover
- `2` (balance-xor): XOR
- `4` (802.3ad): LACP
- `6` (balance-alb): Adaptive load balancing

---

## Pratik Senaryolar

### 1. Network Sorun Giderme

```bash
# 1. Interface aktif mi?
ip link show eth0

# 2. IP adresi var mı?
ip addr show eth0

# 3. Gateway'e ping atabilir miyim?
ping -c 4 192.168.1.1

# 4. DNS çalışıyor mu?
nslookup google.com
dig google.com

# 5. İnternete erişebilir miyim?
ping -c 4 8.8.8.8

# 6. Port açık mı?
sudo ss -ltn | grep :80

# 7. Firewall engel mi?
sudo ufw status
sudo iptables -L
```

---

### 2. Statik IP Ayarlama (Ubuntu)

```bash
sudo nano /etc/netplan/00-installer-config.yaml

network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1

sudo netplan apply
```

---

### 3. Basit Firewall Kurulumu

```bash
# UFW kur ve aktif et
sudo apt install ufw

# Default policy
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH, HTTP, HTTPS
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Belirli IP'den SSH
sudo ufw allow from 192.168.1.100 to any port 22

# Aktif et
sudo ufw enable

# Kontrol
sudo ufw status verbose
```

---

### 4. Port Forwarding (iptables)

```bash
# Port 80'i 8080'e yönlendir
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# Harici IP'ye forward
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.50:80
sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```

---

## 🎯 Özet

| Komut | Açıklama | Örnek |
|-------|----------|-------|
| `ip addr` | IP adresleri | `ip addr show` |
| `ip route` | Routing table | `ip route show` |
| `ping` | Bağlantı testi | `ping google.com` |
| `ss` | Socket istatistikleri | `ss -ltp` |
| `ufw` | Firewall (Ubuntu) | `ufw allow 80` |
| `firewall-cmd` | Firewall (RHEL) | `firewall-cmd --add-service=http` |
| `nmap` | Port scanning | `nmap 192.168.1.1` |
| `tcpdump` | Packet capture | `tcpdump -i eth0` |
| `curl` | HTTP istekleri | `curl https://api.com` |

---

## 🎓 Alıştırmalar

1. Statik IP yapılandır ve test et
2. DNS sunucularını değiştir (Google DNS)
3. Yeni bir route ekle
4. Firewall'da HTTP/HTTPS portlarını aç
5. Ping ile gateway'e ulaşım test et
6. ss ile listening portları listele
7. nmap ile localhost'u tara
8. tcpdump ile HTTP trafiğini yakala
9. curl ile API'ye POST request at
10. Belirli IP'den SSH bağlantısına izin ver

---

**Sonraki Bölüm:** [08-Shell-Scripting.md](08-Shell-Scripting.md) → Bash scripting ve otomasyon
