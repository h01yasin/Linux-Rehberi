# 🐧 LINUX REHBERİ — BÖLÜM 14: PERFORMANCE TUNING

Sistem performans optimizasyonu: CPU, memory, I/O, network tuning, kernel parameters.

---

## Performance Analysis Methodology

```
1. Define problem (Slow? Which part?)
2. Measure performance (Baseline)
3. Identify bottleneck (CPU? RAM? Disk? Network?)
4. Apply fixes
5. Re-measure (Did it improve?)
6. Repeat if needed
```

---

## System Resource Monitoring

### CPU Monitoring

```bash
# Real-time CPU
top
htop

# CPU details
lscpu
cat /proc/cpuinfo

# Per-core usage
mpstat -P ALL 1

# Load average
uptime
cat /proc/loadavg

# Context switches
vmstat 1

# CPU-intensive processes
ps aux --sort=-%cpu | head -10
```

---

### Memory Monitoring

```bash
# Memory usage
free -h
free -m -s 1  # Her saniye

# Memory details
cat /proc/meminfo

# Per-process memory
ps aux --sort=-%mem | head -10

# Detailed memory
sudo smem -tk

# Memory map (belirli process)
sudo pmap -x 1234

# OOM (Out of Memory) logs
dmesg | grep -i "out of memory"
sudo journalctl -k | grep -i "oom"
```

---

### Disk I/O Monitoring

```bash
# I/O statistics
iostat -x 1

# Real-time I/O
sudo iotop

# Detailed disk stats
sudo dstat -d 1

# Per-process I/O
sudo iotop -o

# I/O wait
top  # wa column
```

---

### Network Monitoring

```bash
# Network traffic
sudo iftop

# Bandwidth
nload
bmon

# Connections
ss -s
netstat -s

# Per-process network
sudo nethogs
```

---

## CPU Tuning

### CPU Governor

CPU frekans yönetimi.

```bash
# Mevcut governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Available governors
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors

# Tüm CPU'lar için governor değiştir
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# cpupower ile
sudo apt install linux-tools-common
sudo cpupower frequency-set -g performance
```

**Governor tipleri:**
- `performance`: Her zaman max frekans (sunucular için)
- `powersave`: Her zaman min frekans (laptop'ta pil tasarrufu)
- `ondemand`: Dinamik (yük arttığında frekans artar)
- `conservative`: ondemand'a benzer ama daha yavaş geçiş
- `schedutil`: Scheduler-based (modern)

---

### CPU Affinity

Process'i belirli CPU'ya sabitle.

```bash
# Process'in CPU affinity'si
taskset -cp 1234

# Process'i CPU 0 ve 1'e sabitle
taskset -cp 0,1 1234

# Komut başlatırken belirle
taskset -c 0,1 python app.py

# Multithreaded app için
numactl --physcpubind=0,1 --membind=0 python app.py
```

---

### Process Priority (nice/renice)

```bash
# Priority listesi (-20 en yüksek, 19 en düşük)
ps -eo pid,ni,comm

# Düşük priority ile başlat
nice -n 10 python script.py

# Yüksek priority (root gerekir)
sudo nice -n -10 important_task

# Running process'in priority'sini değiştir
sudo renice -n -5 -p 1234

# Tüm kullanıcının processleri
sudo renice -n 5 -u yasin
```

---

### IRQ Balancing

Interrupt handling CPU'lara dağıt.

```bash
# IRQ balance servisi
sudo systemctl status irqbalance

# Enable/disable
sudo systemctl enable irqbalance
sudo systemctl start irqbalance

# Manual IRQ affinity
# IRQ listesi
cat /proc/interrupts

# IRQ affinity (örnek: IRQ 45'i CPU 2'ye)
echo 4 | sudo tee /proc/irq/45/smp_affinity
# (4 = binary 0100 = CPU 2)
```

---

## Memory Tuning

### Swappiness

Swap kullanım eğilimi (0-100).

```bash
# Mevcut swappiness
cat /proc/sys/vm/swappiness

# Geçici değiştir
sudo sysctl vm.swappiness=10

# Kalıcı
sudo nano /etc/sysctl.conf
```

```
vm.swappiness=10
```

```bash
sudo sysctl -p
```

**Önerilen değerler:**
- `60`: Default
- `10`: Sunucular (daha az swap)
- `1`: Database serverlar (minimize swap)
- `100`: Desktop (aggressive swap)

---

### Transparent Huge Pages (THP)

Büyük memory page'leri (2MB).

```bash
# Durum
cat /sys/kernel/mm/transparent_hugepage/enabled

# Disable (database serverlar için önerilir)
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/enabled

# Kalıcı (GRUB)
sudo nano /etc/default/grub
```

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash transparent_hugepage=never"
```

```bash
sudo update-grub
sudo reboot
```

---

### OOM Killer Tuning

Out-of-memory durumunda hangi process öldürülsün?

```bash
# Process OOM score
cat /proc/1234/oom_score

# OOM adjust (-1000 = never kill, 1000 = first kill)
echo -500 | sudo tee /proc/1234/oom_score_adj

# Systemd service için
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Service]
OOMScoreAdjust=-500
```

---

### Cache Flushing

```bash
# Cache drop (sync önce!)
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches

# 1: PageCache
# 2: dentries and inodes
# 3: PageCache, dentries, inodes
```

---

## Disk I/O Tuning

### I/O Scheduler

```bash
# Mevcut scheduler
cat /sys/block/sda/queue/scheduler

# Değiştir (geçici)
echo mq-deadline | sudo tee /sys/block/sda/queue/scheduler

# Kalıcı (GRUB)
sudo nano /etc/default/grub
```

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=mq-deadline"
```

```bash
sudo update-grub
```

**Scheduler tipleri:**
- `mq-deadline`: Default (çoğu disk için iyi)
- `kyber`: Low-latency
- `bfq`: Desktop (interactive)
- `none`: NVMe için (hardware queue)

---

### Read-ahead

Ne kadar veri önceden okunacak?

```bash
# Mevcut read-ahead (512 byte blocks)
sudo blockdev --getra /dev/sda

# Set read-ahead (8MB = 16384 blocks)
sudo blockdev --setra 16384 /dev/sda

# Kalıcı (udev rule)
sudo nano /etc/udev/rules.d/60-readahead.rules
```

```
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{bdi/read_ahead_kb}="8192"
```

---

### Filesystem Mount Options

```bash
# /etc/fstab tuning
sudo nano /etc/fstab
```

```
# noatime: Access time güncelleme (5-10% performance gain)
UUID=xxx  /  ext4  defaults,noatime  0  1

# nodiratime: Directory access time güncelleme
UUID=xxx  /  ext4  defaults,noatime,nodiratime  0  1

# commit= : Dirty data flush interval (ext4, default 5 sn)
UUID=xxx  /data  ext4  defaults,noatime,commit=60  0  2

# barrier=0 : Write barriers disable (SSD'de gerekebilir, riskli!)
# SADECE UPS varsa!
UUID=xxx  /data  ext4  defaults,noatime,barrier=0  0  2
```

---

### SSD Optimization

```bash
# TRIM support check
lsblk --discard

# TRIM etkin mi?
sudo fstrim -v /

# Otomatik TRIM (fstab)
UUID=xxx  /  ext4  defaults,discard  0  1

# veya periodic TRIM (önerilir)
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer
```

---

## Network Tuning

### Kernel Network Parameters

```bash
sudo nano /etc/sysctl.conf
```

```bash
# TCP buffer sizes
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# Backlog queue
net.core.netdev_max_backlog = 5000

# SYN backlog
net.ipv4.tcp_max_syn_backlog = 8192

# Connection tracking
net.netfilter.nf_conntrack_max = 1048576

# TCP fin timeout
net.ipv4.tcp_fin_timeout = 30

# TCP keepalive
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 60
net.ipv4.tcp_keepalive_probes = 3

# TCP congestion control
net.ipv4.tcp_congestion_control = bbr

# Fast open
net.ipv4.tcp_fastopen = 3

# TCP window scaling
net.ipv4.tcp_window_scaling = 1

# TCP timestamps
net.ipv4.tcp_timestamps = 1

# TIME_WAIT sockets reuse
net.ipv4.tcp_tw_reuse = 1

# Local port range
net.ipv4.ip_local_port_range = 10000 65000
```

```bash
# Uygula
sudo sysctl -p
```

---

### TCP BBR (Congestion Control)

Modern congestion control algorithm.

```bash
# BBR modül yükle
sudo modprobe tcp_bbr
echo "tcp_bbr" | sudo tee -a /etc/modules-load.d/modules.conf

# Aktif et
echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Kontrol et
sysctl net.ipv4.tcp_congestion_control
```

---

### NIC (Network Card) Tuning

```bash
# Ring buffer size
sudo ethtool -g eth0
sudo ethtool -G eth0 rx 4096 tx 4096

# Interrupt coalescing
sudo ethtool -c eth0
sudo ethtool -C eth0 rx-usecs 50 tx-usecs 50

# Offloading
sudo ethtool -k eth0
sudo ethtool -K eth0 tso on gso on gro on

# RSS (Receive Side Scaling)
sudo ethtool -l eth0
sudo ethtool -L eth0 combined 4
```

---

## Application-Level Tuning

### Nginx

```nginx
# /etc/nginx/nginx.conf

# Worker processes (= CPU cores)
worker_processes auto;

# Worker connections
events {
    worker_connections 4096;
    use epoll;
    multi_accept on;
}

http {
    # Keepalive
    keepalive_timeout 65;
    keepalive_requests 100;

    # Sendfile
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    # Buffers
    client_body_buffer_size 128k;
    client_max_body_size 10m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
    output_buffers 1 32k;

    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;

    # Gzip
    gzip on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript;

    # Open file cache
    open_file_cache max=1000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
}
```

---

### Apache

```apache
# /etc/apache2/apache2.conf

# MPM prefork (for mod_php)
<IfModule mpm_prefork_module>
    StartServers             5
    MinSpareServers          5
    MaxSpareServers         10
    MaxRequestWorkers      150
    MaxConnectionsPerChild   0
</IfModule>

# MPM worker (multithreaded)
<IfModule mpm_worker_module>
    StartServers             2
    MinSpareThreads         25
    MaxSpareThreads         75
    ThreadsPerChild         25
    MaxRequestWorkers      150
    MaxConnectionsPerChild   0
</IfModule>

# KeepAlive
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
```

---

### MySQL/MariaDB

```ini
# /etc/mysql/my.cnf

[mysqld]
# InnoDB buffer pool (70-80% of RAM)
innodb_buffer_pool_size = 4G

# InnoDB log
innodb_log_file_size = 512M
innodb_log_buffer_size = 16M

# Query cache (deprecated in 8.0)
query_cache_type = 1
query_cache_size = 128M

# Connections
max_connections = 200

# Thread cache
thread_cache_size = 50

# Table cache
table_open_cache = 2000

# Temp tables
tmp_table_size = 64M
max_heap_table_size = 64M

# Sort/join buffer
sort_buffer_size = 2M
join_buffer_size = 2M
read_buffer_size = 2M

# Slow query log
slow_query_log = 1
long_query_time = 2
```

---

### PostgreSQL

```
# /etc/postgresql/14/main/postgresql.conf

# Memory
shared_buffers = 2GB                # 25% of RAM
effective_cache_size = 6GB          # 50-75% of RAM
work_mem = 16MB
maintenance_work_mem = 512MB

# Connections
max_connections = 200

# WAL
wal_buffers = 16MB
checkpoint_completion_target = 0.9

# Query tuning
random_page_cost = 1.1              # SSD için
effective_io_concurrency = 200      # SSD için

# Autovacuum
autovacuum = on
```

---

## Kernel Parameters (sysctl)

```bash
sudo nano /etc/sysctl.conf
```

**Comprehensive tuning:**
```bash
# === NETWORK ===
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10000 65000

# === MEMORY ===
vm.swappiness = 10
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
vm.vfs_cache_pressure = 50

# === FILE SYSTEM ===
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288

# === KERNEL ===
kernel.pid_max = 4194304
kernel.threads-max = 4194304

# === SECURITY ===
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.tcp_syncookies = 1
```

```bash
sudo sysctl -p
```

---

## Limits (ulimit)

Process resource limits.

```bash
# Mevcut limits
ulimit -a

# Soft/hard limits
ulimit -Sn  # Soft file descriptor limit
ulimit -Hn  # Hard file descriptor limit

# Limits ayarla (geçici)
ulimit -n 65536  # Max open files

# Kalıcı
sudo nano /etc/security/limits.conf
```

```
*               soft    nofile          65536
*               hard    nofile          65536
*               soft    nproc           32768
*               hard    nproc           32768
```

**systemd service için:**
```ini
[Service]
LimitNOFILE=65536
LimitNPROC=32768
```

---

## Performance Profiling

### perf

CPU profiling.

```bash
# Kur
sudo apt install linux-tools-common linux-tools-generic

# System-wide profiling (10 saniye)
sudo perf record -a -g sleep 10

# Report
sudo perf report

# Top (real-time)
sudo perf top

# Flamegraph
sudo perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

---

### strace

System call tracing.

```bash
# Process trace
sudo strace -p 1234

# Summary
sudo strace -c -p 1234

# Belirli syscall
sudo strace -e open,read,write -p 1234

# Time takip
sudo strace -tt -p 1234

# Follow fork
sudo strace -f -p 1234
```

---

### ltrace

Library call tracing.

```bash
# Library calls
ltrace python script.py

# Count calls
ltrace -c python script.py
```

---

## Benchmark Tools

### sysbench

```bash
# Kur
sudo apt install sysbench

# CPU benchmark
sysbench cpu --cpu-max-prime=20000 run

# Memory benchmark
sysbench memory --memory-block-size=1M --memory-total-size=10G run

# File I/O
sysbench fileio --file-total-size=2G prepare
sysbench fileio --file-total-size=2G --file-test-mode=rndrw run
sysbench fileio --file-total-size=2G cleanup

# Database (MySQL)
sysbench oltp_read_write --mysql-user=root --mysql-password=pass prepare
sysbench oltp_read_write --mysql-user=root --mysql-password=pass run
```

---

### Apache Bench (ab)

```bash
# 1000 request, 10 concurrent
ab -n 1000 -c 10 http://localhost/

# Keepalive
ab -n 1000 -c 10 -k http://localhost/
```

---

### wrk

```bash
# Kur
git clone https://github.com/wg/wrk.git
cd wrk
make
sudo cp wrk /usr/local/bin/

# Benchmark
wrk -t12 -c400 -d30s http://localhost/

# Lua script ile
wrk -t12 -c400 -d30s -s post.lua http://localhost/api
```

---

## Best Practices Checklist

### ✅ CPU
- [ ] CPU governor: performance (sunucu için)
- [ ] IRQ balancing aktif
- [ ] High priority process'lere uygun nice value

### ✅ Memory
- [ ] Swappiness ayarlandı (10-20)
- [ ] THP disabled (database için)
- [ ] Uygun OOM score

### ✅ Disk
- [ ] Uygun I/O scheduler (mq-deadline/kyber)
- [ ] noatime mount option
- [ ] SSD'de TRIM aktif
- [ ] Read-ahead optimize

### ✅ Network
- [ ] TCP BBR aktif
- [ ] Kernel network params tuned
- [ ] NIC offloading aktif

### ✅ Application
- [ ] Connection pooling
- [ ] Caching (Redis/Memcached)
- [ ] Keep-alive connections
- [ ] Gzip/Brotli compression

---

## 🎯 Özet

| Alan | Tool | Tuning |
|------|------|--------|
| CPU | top, perf | Governor, affinity, nice |
| Memory | free, vmstat | Swappiness, THP, OOM |
| Disk | iostat, iotop | Scheduler, noatime, TRIM |
| Network | ss, iftop | BBR, sysctl params |
| General | sysctl | Kernel parameters |

---

## 🎓 Alıştırmalar

1. CPU governor'ı performance'a çevir
2. Swappiness'i 10'a ayarla
3. Disk için noatime mount option ekle
4. TCP BBR'ı aktif et
5. Nginx worker_connections'ı artır
6. MySQL innodb_buffer_pool_size ayarla
7. perf ile CPU profiling yap
8. ulimit ile max open files artır
9. sysbench ile CPU benchmark yap
10. sysctl ile network parameters tune et

---

**Sonraki Bölüm:** [15-Backup-ve-Restore.md](15-Backup-ve-Restore.md) → rsync, tar, backup strategies, disaster recovery
