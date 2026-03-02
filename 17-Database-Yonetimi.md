# 🐧 LINUX REHBERİ — BÖLÜM 17: DATABASE YÖNETİMİ

MySQL/MariaDB, PostgreSQL, Redis, backup, replication, optimization.

---

## MySQL / MariaDB

İlişkisel veritabanı (RDBMS). MariaDB = MySQL'in fork'u.

### Kurulum

```bash
# Ubuntu/Debian - MySQL
sudo apt update
sudo apt install mysql-server

# Ubuntu/Debian - MariaDB
sudo apt install mariadb-server

# RHEL/Rocky - MySQL
sudo dnf install mysql-server

# RHEL/Rocky - MariaDB
sudo dnf install mariadb-server

# Start & enable
sudo systemctl enable --now mysql      # MySQL
sudo systemctl enable --now mariadb    # MariaDB

# Secure installation
sudo mysql_secure_installation
```

**mysql_secure_installation:**
- Root password oluştur
- Anonymous user'ları sil
- Remote root login'i kapat
- Test database'i sil

---

### MySQL/MariaDB Bağlanma

```bash
# Root ile bağlan
sudo mysql -u root -p

# Specific database
mysql -u username -p database_name

# Remote host
mysql -h 192.168.1.10 -u username -p

# Execute SQL file
mysql -u root -p < backup.sql

# Execute command
mysql -u root -p -e "SHOW DATABASES;"
```

---

### Database İşlemleri

```sql
-- Database oluştur
CREATE DATABASE myapp;

-- Database listesi
SHOW DATABASES;

-- Database seç
USE myapp;

-- Database sil
DROP DATABASE myapp;

-- Current database
SELECT DATABASE();
```

---

### User Yönetimi

```sql
-- User oluştur
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'strong_password';

-- Remote access için
CREATE USER 'appuser'@'%' IDENTIFIED BY 'strong_password';

-- Specific IP
CREATE USER 'appuser'@'192.168.1.%' IDENTIFIED BY 'strong_password';

-- Privileges ver
GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'localhost';

-- Read-only
GRANT SELECT ON myapp.* TO 'readonly'@'localhost';

-- Specific privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'appuser'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;

-- User listesi
SELECT user, host FROM mysql.user;

-- Privileges göster
SHOW GRANTS FOR 'appuser'@'localhost';

-- Privileges kaldır
REVOKE ALL PRIVILEGES ON myapp.* FROM 'appuser'@'localhost';

-- User sil
DROP USER 'appuser'@'localhost';

-- Password değiştir
ALTER USER 'appuser'@'localhost' IDENTIFIED BY 'new_password';
```

---

### Table İşlemleri

```sql
-- Table oluştur
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table listesi
SHOW TABLES;

-- Table struktur
DESCRIBE users;
SHOW CREATE TABLE users;

-- Data ekleme
INSERT INTO users (username, email) VALUES ('john', 'john@example.com');

-- Data okuma
SELECT * FROM users;
SELECT username, email FROM users WHERE id = 1;

-- Data güncelleme
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- Data silme
DELETE FROM users WHERE id = 1;

-- Table sil
DROP TABLE users;
```

---

### Backup & Restore

#### mysqldump (Logical Backup)

```bash
# Single database backup
mysqldump -u root -p myapp > myapp_backup.sql

# All databases
mysqldump -u root -p --all-databases > all_databases.sql

# Specific tables
mysqldump -u root -p myapp users orders > tables_backup.sql

# With compression
mysqldump -u root -p myapp | gzip > myapp_backup.sql.gz

# Exclude data (structure only)
mysqldump -u root -p --no-data myapp > myapp_structure.sql

# Data only (no structure)
mysqldump -u root -p --no-create-info myapp > myapp_data.sql

# Optimized for large databases
mysqldump -u root -p --single-transaction --quick --lock-tables=false myapp > myapp_backup.sql
```

#### Restore

```bash
# Restore database
mysql -u root -p myapp < myapp_backup.sql

# From compressed backup
gunzip < myapp_backup.sql.gz | mysql -u root -p myapp

# Create database and restore
mysql -u root -p -e "CREATE DATABASE myapp;"
mysql -u root -p myapp < myapp_backup.sql
```

#### Automated Backup Script

```bash
sudo nano /usr/local/bin/mysql-backup.sh
```

```bash
#!/bin/bash
# MySQL Automated Backup Script

# Config
BACKUP_DIR="/backup/mysql"
RETENTION_DAYS=7
MYSQL_USER="root"
MYSQL_PASSWORD="your_password"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Backup all databases
mysqldump -u "$MYSQL_USER" -p"$MYSQL_PASSWORD" --all-databases \
    --single-transaction --quick --lock-tables=false \
    | gzip > "$BACKUP_DIR/all_databases_$DATE.sql.gz"

# Delete old backups
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

# Log
echo "Backup completed: all_databases_$DATE.sql.gz"
```

```bash
chmod +x /usr/local/bin/mysql-backup.sh

# Cron (daily 2 AM)
sudo crontab -e
0 2 * * * /usr/local/bin/mysql-backup.sh >> /var/log/mysql-backup.log 2>&1
```

---

### Replication (Master-Slave)

Master'dan Slave'e otomatik data replikasyonu.

#### Master Config

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf  # Ubuntu
sudo nano /etc/my.cnf.d/server.cnf            # RHEL
```

```ini
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = myapp
```

```bash
sudo systemctl restart mysql

# Master'da user oluştur
sudo mysql -u root -p
```

```sql
CREATE USER 'replicator'@'%' IDENTIFIED BY 'replicator_password';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';
FLUSH PRIVILEGES;

-- Master status (log file ve position not al)
SHOW MASTER STATUS;
```

#### Slave Config

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

```ini
[mysqld]
server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log
```

```bash
sudo systemctl restart mysql

sudo mysql -u root -p
```

```sql
CHANGE MASTER TO
    MASTER_HOST='192.168.1.10',
    MASTER_USER='replicator',
    MASTER_PASSWORD='replicator_password',
    MASTER_LOG_FILE='mysql-bin.000001',  -- SHOW MASTER STATUS'tan al
    MASTER_LOG_POS=123;                  -- SHOW MASTER STATUS'tan al

START SLAVE;

-- Replication status
SHOW SLAVE STATUS\G
```

**Kontroller:**
- `Slave_IO_Running: Yes`
- `Slave_SQL_Running: Yes`
- `Seconds_Behind_Master: 0`

---

### Performance Tuning

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

```ini
[mysqld]
# InnoDB buffer pool (75% of RAM for dedicated DB server)
innodb_buffer_pool_size = 2G

# Max connections
max_connections = 200

# Query cache (deprecated in MySQL 8.0)
query_cache_type = 1
query_cache_size = 64M

# Temp table
tmp_table_size = 64M
max_heap_table_size = 64M

# Logging
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow-query.log
long_query_time = 2

# Binlog
expire_logs_days = 7
max_binlog_size = 100M
```

```bash
sudo systemctl restart mysql
```

---

### Monitoring

```sql
-- Process list
SHOW PROCESSLIST;

-- Status variables
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Uptime';

-- Table sizes
SELECT 
    table_schema AS 'Database',
    table_name AS 'Table',
    ROUND((data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.TABLES
ORDER BY (data_length + index_length) DESC
LIMIT 10;

-- Slow queries
SELECT * FROM mysql.slow_log ORDER BY query_time DESC LIMIT 10;
```

---

## PostgreSQL

Enterprise-grade açık kaynak RDBMS.

### Kurulum

```bash
# Ubuntu/Debian
sudo apt install postgresql postgresql-contrib

# RHEL/Rocky
sudo dnf install postgresql-server postgresql-contrib
sudo postgresql-setup --initdb

# Start & enable
sudo systemctl enable --now postgresql

# Version
psql --version
```

---

### PostgreSQL Bağlanma

```bash
# postgres user ile bağlan
sudo -u postgres psql

# Specific database
sudo -u postgres psql -d myapp

# Remote host
psql -h 192.168.1.10 -U username -d database

# Exit
\q
```

---

### Database İşlemleri

```sql
-- Database oluştur
CREATE DATABASE myapp;

-- Database listesi
\l

-- Database seç
\c myapp

-- Database sil
DROP DATABASE myapp;

-- Current database
SELECT current_database();
```

---

### User Yönetimi

```sql
-- User oluştur
CREATE USER appuser WITH PASSWORD 'strong_password';

-- Superuser
CREATE USER admin WITH SUPERUSER PASSWORD 'strong_password';

-- Database privileges
GRANT ALL PRIVILEGES ON DATABASE myapp TO appuser;

-- Table privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO appuser;

-- User listesi
\du

-- Password değiştir
ALTER USER appuser WITH PASSWORD 'new_password';

-- User sil
DROP USER appuser;
```

---

### Table İşlemleri

```sql
-- Table oluştur
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table listesi
\dt

-- Table structure
\d users

-- Data ekleme
INSERT INTO users (username, email) VALUES ('john', 'john@example.com');

-- Data okuma
SELECT * FROM users;

-- Data güncelleme
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- Data silme
DELETE FROM users WHERE id = 1;
```

---

### Backup & Restore

```bash
# Database backup
sudo -u postgres pg_dump myapp > myapp_backup.sql

# Compressed backup
sudo -u postgres pg_dump myapp | gzip > myapp_backup.sql.gz

# All databases
sudo -u postgres pg_dumpall > all_databases.sql

# Custom format (faster restore)
sudo -u postgres pg_dump -Fc myapp > myapp_backup.dump

# Restore
sudo -u postgres psql myapp < myapp_backup.sql

# Restore from custom format
sudo -u postgres pg_restore -d myapp myapp_backup.dump

# Restore with create database
sudo -u postgres pg_restore -C -d postgres myapp_backup.dump
```

**Automated Backup:**
```bash
sudo nano /usr/local/bin/pg-backup.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/backup/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

sudo -u postgres pg_dumpall | gzip > "$BACKUP_DIR/all_databases_$DATE.sql.gz"

# Retention
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete

echo "Backup completed: $DATE"
```

```bash
chmod +x /usr/local/bin/pg-backup.sh

# Cron
sudo crontab -e
0 2 * * * /usr/local/bin/pg-backup.sh >> /var/log/pg-backup.log 2>&1
```

---

### Remote Access

**postgresql.conf:**
```bash
sudo nano /etc/postgresql/14/main/postgresql.conf  # Ubuntu
sudo nano /var/lib/pgsql/data/postgresql.conf      # RHEL
```

```ini
listen_addresses = '*'
port = 5432
max_connections = 100
```

**pg_hba.conf:**
```bash
sudo nano /etc/postgresql/14/main/pg_hba.conf
```

```
# TYPE  DATABASE  USER      ADDRESS          METHOD
local   all       postgres                   peer
host    all       all       127.0.0.1/32     md5
host    all       all       192.168.1.0/24   md5
```

```bash
sudo systemctl restart postgresql
```

---

### Replication (Primary-Standby)

#### Primary Config

```bash
sudo nano /etc/postgresql/14/main/postgresql.conf
```

```ini
wal_level = replica
max_wal_senders = 3
wal_keep_size = 64
```

```sql
-- Replication user oluştur
CREATE USER replicator WITH REPLICATION PASSWORD 'replicator_password';
```

**pg_hba.conf:**
```
host    replication    replicator    192.168.1.0/24    md5
```

```bash
sudo systemctl restart postgresql
```

#### Standby Config

```bash
# Base backup al
sudo -u postgres pg_basebackup -h 192.168.1.10 -D /var/lib/postgresql/14/main -U replicator -P --wal-method=stream

# Standby signal
sudo -u postgres touch /var/lib/postgresql/14/main/standby.signal

# Config
sudo nano /var/lib/postgresql/14/main/postgresql.auto.conf
```

```ini
primary_conninfo = 'host=192.168.1.10 port=5432 user=replicator password=replicator_password'
```

```bash
sudo systemctl start postgresql
```

**Kontrol:**
```sql
-- Primary'de
SELECT * FROM pg_stat_replication;

-- Standby'de
SELECT pg_is_in_recovery();
```

---

### Performance Tuning

```ini
# /etc/postgresql/14/main/postgresql.conf

# Memory
shared_buffers = 256MB           # 25% of RAM
effective_cache_size = 1GB       # 50-75% of RAM
work_mem = 16MB
maintenance_work_mem = 128MB

# WAL
wal_buffers = 16MB
checkpoint_completion_target = 0.9

# Planner
random_page_cost = 1.1           # For SSD (default 4.0)
effective_io_concurrency = 200

# Logging
log_min_duration_statement = 1000  # Log queries > 1s
```

---

## Redis

In-memory NoSQL key-value store. Cache, session, queue.

### Kurulum

```bash
# Ubuntu/Debian
sudo apt install redis-server

# RHEL/Rocky
sudo dnf install redis

# Start & enable
sudo systemctl enable --now redis

# Test
redis-cli ping
```

---

### redis.conf

```bash
sudo nano /etc/redis/redis.conf
```

```ini
# Bind
bind 127.0.0.1 ::1

# Remote access için
# bind 0.0.0.0

# Port
port 6379

# Password
requirepass your_redis_password

# Persistence (RDB)
save 900 1       # 900 saniyede 1 key değişirse kaydet
save 300 10
save 60 10000

# Max memory
maxmemory 256mb
maxmemory-policy allkeys-lru
```

```bash
sudo systemctl restart redis
```

---

### Redis CLI

```bash
# Bağlan
redis-cli

# Password ile
redis-cli -a your_redis_password

# Remote host
redis-cli -h 192.168.1.10 -p 6379 -a password

# Test
PING
```

**String ops:**
```redis
SET key value
GET key
SET name "John Doe"
GET name
INCR counter
DECR counter
DEL key
EXISTS key
EXPIRE key 60          # 60 saniye sonra sil
TTL key                # Remaining time
```

**List ops:**
```redis
LPUSH mylist "item1"
LPUSH mylist "item2"
RPUSH mylist "item3"
LRANGE mylist 0 -1     # All items
LPOP mylist            # Remove first
RPOP mylist            # Remove last
```

**Hash ops:**
```redis
HSET user:1 name "John"
HSET user:1 email "john@example.com"
HGET user:1 name
HGETALL user:1
HDEL user:1 email
```

**Set ops:**
```redis
SADD tags "linux"
SADD tags "devops"
SMEMBERS tags
SISMEMBER tags "linux"
```

---

### Redis Backup

```bash
# Manual backup (creates dump.rdb)
redis-cli BGSAVE

# Check backup status
redis-cli LASTSAVE

# Backup file location
/var/lib/redis/dump.rdb

# Automated backup
sudo crontab -e
0 2 * * * cp /var/lib/redis/dump.rdb /backup/redis/dump-$(date +\%Y\%m\%d).rdb
```

---

### Redis Monitoring

```redis
INFO                   # All info
INFO stats             # Stats
INFO memory            # Memory usage
INFO replication       # Replication info

MONITOR                # Real-time commands (debug only)
SLOWLOG GET 10         # Slow queries

CLIENT LIST            # Connected clients
```

---

## Database Comparison

| Feature | MySQL | PostgreSQL | Redis |
|---------|-------|------------|-------|
| Type | RDBMS | RDBMS | NoSQL (key-value) |
| ACID | ✅ Yes | ✅ Yes | ⚠️ Limited |
| Performance | 🟢 Fast | 🟢 Fast | ⚡ Very Fast (in-memory) |
| Scalability | 🟢 Good | 🟢 Good | ⚡ Excellent |
| Replication | ✅ Master-Slave | ✅ Primary-Standby | ✅ Master-Replica |
| Use Case | Web apps, WordPress | Enterprise, Complex queries | Cache, Session, Queue |
| Learning Curve | 🟢 Easy | 🟡 Medium | 🟢 Easy |

---

## 🎯 Özet

| Database | Install | Connect | Backup |
|----------|---------|---------|--------|
| MySQL | `apt install mysql-server` | `mysql -u root -p` | `mysqldump myapp > backup.sql` |
| PostgreSQL | `apt install postgresql` | `sudo -u postgres psql` | `pg_dump myapp > backup.sql` |
| Redis | `apt install redis-server` | `redis-cli` | `redis-cli BGSAVE` |

---

## 🎓 Alıştırmalar

1. MySQL kur, database oluştur, user ekle
2. mysqldump ile backup al, restore et
3. MySQL slow query log analizi
4. MySQL master-slave replication kur
5. PostgreSQL kur, database oluştur
6. pg_dump ile backup al
7. PostgreSQL primary-standby replication
8. Redis kur, password ayarla
9. Redis cache örneği (web app ile)
10. Automated backup script yaz (3 database için)

---

**Sonraki Bölüm:** [18-Ansible.md](18-Ansible.md) → Configuration Management, Infrastructure as Code
