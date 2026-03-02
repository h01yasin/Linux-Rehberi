# 🐧 LINUX REHBERİ — BÖLÜM 12: LOGGING VE MONİTORİNG

rsyslog, logrotate, systemd-journald, monitoring tools (Prometheus, Grafana, Zabbix).

---

## Linux Logging Sistemi

```
┌─────────────────────────────────────────┐
│         Applications                     │
│    (nginx, mysql, app.py)                │
└──────────────┬──────────────────────────┘
               │
       ┌───────┴──────────┐
       ▼                  ▼
┌─────────────┐   ┌──────────────────┐
│  rsyslog    │   │ systemd-journald │
│ /var/log/*  │   │   journalctl     │
└─────────────┘   └──────────────────┘
       │                  │
       └────────┬─────────┘
                ▼
       ┌────────────────┐
       │   logrotate    │
       │  (rotation)    │
       └────────────────┘
```

---

## rsyslog

Geleneksel syslog daemon.

### Ayarlar

```bash
# Main config
sudo nano /etc/rsyslog.conf

# Additional configs
ls /etc/rsyslog.d/
```

---

### rsyslog Syntax

```
facility.priority    action
```

**Facilities:**
```
auth, authpriv   # Authentication
cron             # Cron daemon
daemon           # System daemons
kern             # Kernel
mail             # Mail system
user             # User processes
local0-local7    # Custom use
*                # All facilities
```

**Priorities:**
```
emerg (0)   # Emergency
alert (1)   # Alert
crit (2)    # Critical
err (3)     # Error
warning (4) # Warning
notice (5)  # Notice
info (6)    # Info
debug (7)   # Debug
*           # All priorities
```

---

### rsyslog Örnekler

```bash
# /etc/rsyslog.conf

# Tüm loglar
*.*                          /var/log/all.log

# Auth logları
auth,authpriv.*              /var/log/auth.log

# Kernel logları
kern.*                       /var/log/kern.log

# Mail logları
mail.*                       /var/log/mail.log

# Cron logları
cron.*                       /var/log/cron.log

# Emergency herkese
*.emerg                      :omusrmsg:*

# Error ve üzeri remote server'a
*.err                        @@remote-server:514

# Custom app
local0.*                     /var/log/myapp.log

# Exclude (!)
*.* ;kern.none               /var/log/messages
```

---

### Remote Logging

**Server (log alıcı):**
```bash
sudo nano /etc/rsyslog.conf

# UDP
$ModLoad imudp
$UDPServerRun 514

# TCP (önerilir)
$ModLoad imtcp
$InputTCPServerRun 514
```

**Client (log gönderen):**
```bash
sudo nano /etc/rsyslog.conf

# UDP
*.* @remote-server:514

# TCP
*.* @@remote-server:514
```

```bash
# Restart
sudo systemctl restart rsyslog
```

---

### Custom Log Dosyası

```bash
# Config
sudo nano /etc/rsyslog.d/myapp.conf
```

```
# MyApp logs
if $programname == 'myapp' then /var/log/myapp.log
& stop
```

```bash
# Restart
sudo systemctl restart rsyslog

# Test (logger komutu)
logger -t myapp "Test message"
tail /var/log/myapp.log
```

---

## logrotate

Log dosyalarını döndürme, sıkıştırma, silme.

### Config

```bash
# Main config
sudo nano /etc/logrotate.conf

# App configs
ls /etc/logrotate.d/
```

---

### logrotate Sözdizimi

```bash
/var/log/myapp.log {
    daily                # Günlük rotate
    rotate 7             # 7 adet backup tut
    compress             # Eski logları sıkıştır
    delaycompress        # En son'u sıkıştırma
    missingok            # Dosya yoksa hata verme
    notifempty           # Boşsa rotate yapma
    create 0640 root utmp  # Yeni dosya permissions
    sharedscripts        # Pre/post script bir kez çalışsın
    postrotate
        systemctl reload myapp
    endscript
}
```

---

### logrotate Örnekler

**1. Nginx:**
```bash
sudo nano /etc/logrotate.d/nginx
```

```
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        if [ -f /var/run/nginx.pid ]; then
            kill -USR1 $(cat /var/run/nginx.pid)
        fi
    endscript
}
```

**2. Custom app:**
```bash
sudo nano /etc/logrotate.d/myapp
```

```
/var/log/myapp/*.log {
    weekly
    rotate 4
    compress
    delaycompress
    missingok
    notifempty
    create 0644 myapp myapp
    postrotate
        systemctl reload myapp
    endscript
}
```

**3. Size-based rotation:**
```
/var/log/myapp.log {
    size 100M
    rotate 5
    compress
    missingok
    notifempty
}
```

---

### logrotate Manuel Çalıştırma

```bash
# Dry run (test)
sudo logrotate -d /etc/logrotate.conf

# Force run
sudo logrotate -f /etc/logrotate.conf

# Belirli config
sudo logrotate -f /etc/logrotate.d/myapp

# Verbose
sudo logrotate -v /etc/logrotate.conf

# Status
cat /var/lib/logrotate/status
```

---

### logrotate Cron

```bash
# Günlük çalışır (default)
cat /etc/cron.daily/logrotate
```

---

## systemd-journald

Systemd'nin binary log sistemi.

### journalctl (Zaten 10. bölümde gördük, özet)

```bash
# Son 100 satır
journalctl -n 100

# Canlı takip
journalctl -f

# Belirli servis
journalctl -u nginx

# Priority filter
journalctl -p err

# Zaman filtreleme
journalctl --since today
journalctl --since "2024-01-15 10:00" --until "2024-01-15 12:00"

# Boot logs
journalctl -b
journalctl -b -1  # Önceki boot

# Kernel logs
journalctl -k

# Specific user
journalctl _UID=1000

# JSON output
journalctl -o json

# Disk kullanımı
journalctl --disk-usage

# Vacuum (temizlik)
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=100M
```

---

### journald Config

```bash
sudo nano /etc/systemd/journald.conf
```

```ini
[Journal]
# Storage
Storage=persistent        # /var/log/journal (kalıcı)
Storage=volatile         # /run/log/journal (RAM)
Storage=auto             # Varsa persistent

# Size limits
SystemMaxUse=500M        # Max disk kullanımı
SystemMaxFileSize=50M    # Max dosya boyutu
RuntimeMaxUse=100M       # Max RAM kullanımı

# Retention
MaxRetentionSec=1month   # Max saklama süresi
MaxFileSec=1week         # Max dosya yaşı

# Rate limiting
RateLimitIntervalSec=30s
RateLimitBurst=10000

# Forward to syslog
ForwardToSyslog=yes
ForwardToWall=no
```

```bash
sudo systemctl restart systemd-journald
```

---

## Application Logging

### Python (logging)

```python
import logging
import logging.handlers

# Basic config
logging.basicConfig(level=logging.INFO, 
                    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')

logger = logging.getLogger('myapp')

# Syslog handler
syslog_handler = logging.handlers.SysLogHandler(address='/dev/log')
logger.addHandler(syslog_handler)

# File handler
file_handler = logging.FileHandler('/var/log/myapp.log')
file_handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)

# Rotating file handler
rotating_handler = logging.handlers.RotatingFileHandler(
    '/var/log/myapp.log',
    maxBytes=10*1024*1024,  # 10MB
    backupCount=5
)
logger.addHandler(rotating_handler)

# Usage
logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
logger.critical("Critical message")
```

---

### Nginx/Apache Logs

**Nginx:**
```bash
# access.log
tail -f /var/log/nginx/access.log

# error.log
tail -f /var/log/nginx/error.log

# Custom log format
# /etc/nginx/nginx.conf
log_format custom '$remote_addr - $remote_user [$time_local] '
                  '"$request" $status $body_bytes_sent '
                  '"$http_referer" "$http_user_agent" '
                  '$request_time';

access_log /var/log/nginx/access.log custom;
```

**Apache:**
```bash
# access.log
tail -f /var/log/apache2/access.log

# error.log
tail -f /var/log/apache2/error.log
```

---

## Monitoring Tools

### 1. Htop (Process Monitoring)

```bash
sudo apt install htop
htop
```

---

### 2. Glances (System Monitoring)

```bash
# Kur
sudo apt install glances

# Çalıştır
glances

# Web mode
glances -w
# http://localhost:61208

# Client/Server mode
# Server
glances -s

# Client
glances -c <server_ip>
```

---

### 3. Netdata (Real-time Monitoring)

```bash
# Kur
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# Web interface
# http://localhost:19999

# Config
sudo nano /etc/netdata/netdata.conf

# Restart
sudo systemctl restart netdata
```

**Features:**
- CPU, RAM, Disk, Network
- Per-process monitoring
- Web metrics
- Alerts
- Plugin system

---

### 4. Prometheus + Grafana

Modern monitoring stack.

#### Prometheus (Metrics Collector)

```bash
# Download
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
cd prometheus-*

# Config
nano prometheus.yml
```

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'nginx'
    static_configs:
      - targets: ['localhost:9113']
```

```bash
# Run
./prometheus --config.file=prometheus.yml

# Web UI: http://localhost:9090
```

---

#### Node Exporter (System Metrics)

```bash
# Download
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
cd node_exporter-*

# Run
./node_exporter

# Metrics: http://localhost:9100/metrics
```

---

#### Grafana (Visualization)

```bash
# Kur (Ubuntu)
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana

# Start
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Web UI: http://localhost:3000
# Default: admin/admin
```

**Grafana Setup:**
1. Add data source: Prometheus → http://localhost:9090
2. Import dashboard: Node Exporter Full (ID: 1860)
3. Create custom dashboards

---

### 5. Zabbix

Enterprise monitoring solution.

```bash
# Kur (Ubuntu)
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update

# Server, frontend, agent
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-agent

# Database
sudo apt install mysql-server
sudo mysql -uroot -p
```

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

```bash
# Import schema
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

# Config
sudo nano /etc/zabbix/zabbix_server.conf
```

```
DBPassword=password
```

```bash
# Start
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2

# Web: http://localhost/zabbix
# Default: Admin/zabbix
```

---

### 6. ELK Stack (Elasticsearch, Logstash, Kibana)

Log aggregation ve analiz.

**Elasticsearch:**
```bash
# Kur
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
sudo apt install elasticsearch

# Start
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch

# Test
curl -X GET "localhost:9200"
```

**Logstash:**
```bash
# Kur
sudo apt install logstash

# Config
sudo nano /etc/logstash/conf.d/syslog.conf
```

```
input {
  file {
    path => "/var/log/syslog"
    type => "syslog"
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGBASE}" }
    }
    date {
      match => [ "timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "syslog-%{+YYYY.MM.dd}"
  }
}
```

```bash
# Start
sudo systemctl start logstash
sudo systemctl enable logstash
```

**Kibana:**
```bash
# Kur
sudo apt install kibana

# Config
sudo nano /etc/kibana/kibana.yml
```

```yaml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
```

```bash
# Start
sudo systemctl start kibana
sudo systemctl enable kibana

# Web: http://localhost:5601
```

---

## Alerting

### 1. Email Alerts (mailx)

```bash
# Kur
sudo apt install mailutils

# Test
echo "Test email body" | mail -s "Test Subject" user@example.com

# Script
#!/bin/bash
DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}' | sed 's/%//')

if [ $DISK_USAGE -gt 80 ]; then
    echo "Disk usage is ${DISK_USAGE}%" | mail -s "Disk Alert" admin@example.com
fi
```

---

### 2. systemd Service Alerts

```bash
sudo nano /etc/systemd/system/alert-on-failure@.service
```

```ini
[Unit]
Description=Alert on %i failure

[Service]
Type=oneshot
ExecStart=/usr/local/bin/alert-failure.sh %i
```

```bash
sudo nano /usr/local/bin/alert-failure.sh
```

```bash
#!/bin/bash
SERVICE=$1
echo "Service $SERVICE failed!" | mail -s "Service Alert" admin@example.com
```

```bash
chmod +x /usr/local/bin/alert-failure.sh

# Service'e ekle
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
OnFailure=alert-on-failure@%n.service
```

---

### 3. Prometheus Alertmanager

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@example.com'
  smtp_auth_username: 'alerts@example.com'
  smtp_auth_password: 'password'

route:
  receiver: 'email'

receivers:
  - name: 'email'
    email_configs:
      - to: 'admin@example.com'
```

**Prometheus rules:**
```yaml
# alert.rules.yml
groups:
  - name: example
    rules:
      - alert: HighCPU
        expr: node_cpu_seconds_total > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
```

---

## Log Analysis

### 1. grep/awk Kombinasyonları

```bash
# En çok erişen IP'ler (nginx)
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# 404 hataları
awk '$9 == 404 {print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Belirli saat aralığı
awk '/15:00/,/16:00/ {print}' /var/log/syslog

# Error count per hour
grep "error" /var/log/app.log | awk '{print $1, $2}' | cut -d: -f1 | sort | uniq -c
```

---

### 2. GoAccess (Web Log Analyzer)

```bash
# Kur
sudo apt install goaccess

# Terminal
goaccess /var/log/nginx/access.log

# HTML report
goaccess /var/log/nginx/access.log -o report.html

# Real-time HTML
goaccess /var/log/nginx/access.log -o /var/www/html/report.html --real-time-html

# Custom log format
goaccess /var/log/nginx/access.log --log-format=COMBINED
```

---

## Praktik Monitoring Script

```bash
#!/bin/bash
# system-monitor.sh

HOSTNAME=$(hostname)
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

# CPU
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)

# Memory
MEM_TOTAL=$(free -m | awk 'NR==2{print $2}')
MEM_USED=$(free -m | awk 'NR==2{print $3}')
MEM_PERCENT=$(echo "scale=2; $MEM_USED / $MEM_TOTAL * 100" | bc)

# Disk
DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}' | sed 's/%//')

# Load average
LOAD=$(uptime | awk -F'load average:' '{print $2}')

echo "===== SYSTEM REPORT: $HOSTNAME ====="
echo "Time: $TIMESTAMP"
echo
echo "CPU Usage: ${CPU_USAGE}%"
echo "Memory Usage: ${MEM_USED}MB / ${MEM_TOTAL}MB (${MEM_PERCENT}%)"
echo "Disk Usage: ${DISK_USAGE}%"
echo "Load Average:$LOAD"
echo

# Alerts
if (( $(echo "$CPU_USAGE > 80" | bc -l) )); then
    echo "⚠️  HIGH CPU USAGE!"
fi

if (( $(echo "$MEM_PERCENT > 80" | bc -l) )); then
    echo "⚠️  HIGH MEMORY USAGE!"
fi

if [ $DISK_USAGE -gt 80 ]; then
    echo "⚠️  HIGH DISK USAGE!"
fi
```

---

## 🎯 Özet

| Tool | Amaç | Kullanım |
|------|------|----------|
| rsyslog | System logging | `/var/log/*` |
| logrotate | Log rotation | `/etc/logrotate.d/*` |
| journalctl | systemd logs | `journalctl -u nginx` |
| Prometheus | Metrics collector | Time-series DB |
| Grafana | Visualization | Dashboards |
| Netdata | Real-time monitoring | Web UI |
| ELK Stack | Log aggregation | Elasticsearch + Kibana |
| Zabbix | Enterprise monitoring | Full solution |

---

## 🎓 Alıştırmalar

1. Custom app için rsyslog config oluştur
2. logrotate ile haftalık rotation ayarla
3. journald'de 7 günden eski logları sil
4. Python app'e logging ekle
5. Prometheus + Node Exporter kur
6. Grafana'da custom dashboard oluştur
7. Disk kullanımı %80'i geçince email gönder
8. nginx access.log'u GoAccess ile analiz et
9. systemd service için failure alert kur
10. System monitoring script yaz ve cron'a ekle

---

**Sonraki Bölüm:** [13-Disk-Depolama.md](13-Disk-Depolama.md) → LVM, RAID, mount, fstab, disk management
