# 🐧 LINUX REHBERİ — BÖLÜM 16: WEB SERVERLERİ

Nginx, Apache, SSL/TLS, reverse proxy, load balancing, virtual hosts.

---

## Nginx

Hafif, yüksek performanslı web server ve reverse proxy.

### Kurulum

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nginx

# RHEL/Rocky
sudo dnf install nginx

# Start & enable
sudo systemctl enable --now nginx

# Test
curl localhost
```

---

### Nginx Yapısı

```
/etc/nginx/
├── nginx.conf           # Ana config
├── sites-available/     # Site configs (Ubuntu)
├── sites-enabled/       # Aktif siteler (symlink)
├── conf.d/              # Additional configs (RHEL)
└── snippets/            # Reusable configs

/var/www/html/           # Default web root
/var/log/nginx/          # Logs
```

---

### nginx.conf

```bash
sudo nano /etc/nginx/nginx.conf
```

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log warn;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    # Logging
    access_log /var/log/nginx/access.log;
    
    # MIME types
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Optimization
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    
    # Timeouts
    keepalive_timeout 65;
    
    # Gzip
    gzip on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # Virtual hosts
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

### Virtual Host (Server Block)

```bash
sudo nano /etc/nginx/sites-available/example.com
```

```nginx
server {
    listen 80;
    listen [::]:80;
    
    server_name example.com www.example.com;
    
    root /var/www/example.com;
    index index.html index.htm;
    
    # Logs
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;
    
    # Main location
    location / {
        try_files $uri $uri/ =404;
    }
    
    # Static files cache
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 365d;
        add_header Cache-Control "public, immutable";
    }
    
    # Hide hidden files
    location ~ /\. {
        deny all;
    }
}
```

```bash
# Web root oluştur
sudo mkdir -p /var/www/example.com
echo "<h1>Welcome to example.com</h1>" | sudo tee /var/www/example.com/index.html

# Enable site (Ubuntu)
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

# Test config
sudo nginx -t

# Reload
sudo systemctl reload nginx
```

---

### PHP-FPM ile

```bash
# PHP-FPM kur
sudo apt install php-fpm

# Nginx config
sudo nano /etc/nginx/sites-available/example.com
```

```nginx
server {
    listen 80;
    server_name example.com;
    
    root /var/www/example.com;
    index index.php index.html;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    # PHP processing
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    # Deny access to .php files in /uploads/
    location ~* /uploads/.*\.php$ {
        deny all;
    }
}
```

---

### Reverse Proxy

Backend app'e proxy.

```nginx
server {
    listen 80;
    server_name app.example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        
        # Headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

---

### Load Balancing

```nginx
# Upstream definition
upstream backend {
    # Load balancing method
    least_conn;  # or: round_robin (default), ip_hash, hash $request_uri
    
    server backend1.example.com:8080 weight=3;
    server backend2.example.com:8080 weight=2;
    server backend3.example.com:8080 weight=1 backup;
    
    keepalive 32;
}

server {
    listen 80;
    server_name app.example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

**Load balancing methods:**
- `round_robin`: Sırayla (default)
- `least_conn`: En az bağlantı
- `ip_hash`: IP'ye göre (session persistence)
- `hash $variable`: Custom key

---

### SSL/TLS (HTTPS)

#### Let's Encrypt (Free SSL)

```bash
# Certbot kur
sudo apt install certbot python3-certbot-nginx

# SSL certificate al
sudo certbot --nginx -d example.com -d www.example.com

# Test renewal
sudo certbot renew --dry-run

# Auto-renewal (cron)
sudo systemctl status certbot.timer
```

**Manuel SSL config:**
```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    
    server_name example.com;
    
    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # SSL config
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    root /var/www/example.com;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}

# HTTP to HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}
```

---

### Rate Limiting

Brute-force, DDoS koruması.

```nginx
# nginx.conf (http block)
http {
    # Zone definition (10 req/sec per IP)
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
    
    # Connection limit
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    
    server {
        location / {
            # Apply rate limit
            limit_req zone=one burst=20 nodelay;
            
            # Connection limit (max 10 per IP)
            limit_conn addr 10;
        }
        
        location /api/ {
            # Stricter limit for API
            limit_req zone=one burst=5 nodelay;
        }
    }
}
```

---

### Access Control

```nginx
server {
    location /admin/ {
        # IP whitelist
        allow 192.168.1.0/24;
        allow 10.0.0.1;
        deny all;
        
        # Basic auth
        auth_basic "Admin Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

**htpasswd oluştur:**
```bash
sudo apt install apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd admin
```

---

## Apache

Modüler, esnek web server.

### Kurulum

```bash
# Ubuntu/Debian
sudo apt install apache2

# RHEL/Rocky
sudo dnf install httpd

# Start & enable
sudo systemctl enable --now apache2  # Ubuntu
sudo systemctl enable --now httpd    # RHEL

# Test
curl localhost
```

---

### Apache Yapısı

```
/etc/apache2/           # Ubuntu
/etc/httpd/             # RHEL

apache2.conf / httpd.conf  # Ana config
sites-available/           # Site configs (Ubuntu)
sites-enabled/             # Aktif siteler
mods-available/            # Modüller
mods-enabled/              # Aktif modüller
ports.conf                 # Port config

/var/www/html/             # Default web root
/var/log/apache2/          # Logs (Ubuntu)
/var/log/httpd/            # Logs (RHEL)
```

---

### Virtual Host

```bash
sudo nano /etc/apache2/sites-available/example.com.conf
```

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin admin@example.com
    
    DocumentRoot /var/www/example.com
    
    <Directory /var/www/example.com>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined
</VirtualHost>
```

```bash
# Enable site
sudo a2ensite example.com

# Disable site
sudo a2dissite 000-default

# Test config
sudo apache2ctl configtest

# Reload
sudo systemctl reload apache2
```

---

### PHP (mod_php)

```bash
# PHP kur
sudo apt install php libapache2-mod-php

# Test
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php

# Reload
sudo systemctl reload apache2

# Test: http://localhost/info.php
```

---

### .htaccess

Directory-level config.

```bash
# .htaccess enable
# /etc/apache2/sites-available/example.com.conf
<Directory /var/www/example.com>
    AllowOverride All
</Directory>
```

**Örnek .htaccess:**
```apache
# Redirect HTTP to HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# Disable directory listing
Options -Indexes

# Custom error pages
ErrorDocument 404 /404.html
ErrorDocument 500 /500.html

# Password protection
AuthType Basic
AuthName "Protected Area"
AuthUserFile /var/ www/.htpasswd
Require valid-user

# Deny access to files
<Files "config.php">
    Require all denied
</Files>

# Cache control
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
</IfModule>
```

---

### Apache Modules

```bash
# Module listesi
apache2ctl -M

# Enable module
sudo a2enmod rewrite
sudo a2enmod ssl
sudo a2enmod headers
sudo a2enmod proxy
sudo a2enmod proxy_http

# Disable module
sudo a2dismod status

# Reload
sudo systemctl reload apache2
```

---

### SSL/TLS

```bash
# SSL module enable
sudo a2enmod ssl

# Certificate oluştur (self-signed)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/apache-selfsigned.key \
    -out /etc/ssl/certs/apache-selfsigned.crt

# Virtual host
sudo nano /etc/apache2/sites-available/example.com-ssl.conf
```

```apache
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
    
    # Security
    Header always set Strict-Transport-Security "max-age=31536000"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
</VirtualHost>

<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

```bash
sudo a2ensite example.com-ssl
sudo systemctl reload apache2
```

---

### Reverse Proxy (Apache)

```bash
# Modules enable
sudo a2enmod proxy
sudo a2enmod proxy_http

# Config
sudo nano /etc/apache2/sites-available/app.conf
```

```apache
<VirtualHost *:80>
    ServerName app.example.com
    
    ProxyPreserveHost On
    ProxyPass / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/
    
    # Headers
    RequestHeader set X-Forwarded-Proto "http"
    RequestHeader set X-Forwarded-Port "80"
</VirtualHost>
```

---

## Web Server Comparison

| Feature | Nginx | Apache |
|---------|-------|--------|
| Architecture | Event-driven | Process-based |
| Performance | ⚡ Çok yüksek | ⚖️ İyi |
| Memory | 💚 Az | 🟡 Orta |
| Static files | ⚡ Mükemmel | ⚖️ İyi |
| Dynamic content | Via proxy | Native (mod_php) |
| Config | Declarative | Procedural |
| .htaccess | ❌ Yok | ✅ Var |
| Modules | Built-in | Loadable |
| Learning curve | Kolay | Orta |

**Kullanım senaryoları:**
- **Nginx**: High traffic, static files, reverse proxy, load balancer
- **Apache**: Legacy apps, .htaccess gerekiyorsa, mod_php

---

## Performance Tuning

### Nginx Tuning

```nginx
# nginx.conf
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    worker_connections 4096;
    use epoll;
    multi_accept on;
}

http {
    # Sendfile
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    
    # Buffers
    client_body_buffer_size 128k;
    client_max_body_size 10m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
    
    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
    keepalive_timeout 15;
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

### Apache Tuning

```apache
# /etc/apache2/mods-available/mpm_prefork.conf
<IfModule mpm_prefork_module>
    StartServers             5
    MinSpareServers          5
    MaxSpareServers         10
    MaxRequestWorkers      150
    MaxConnectionsPerChild 3000
</IfModule>

# KeepAlive
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

# Compression
LoadModule deflate_module modules/mod_deflate.so
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css application/javascript
</IfModule>
```

---

## 🎯 Özet

| Web Server | Kur | Config | Reload |
|-----------|-----|--------|--------|
| Nginx | `apt install nginx` | `/etc/nginx/nginx.conf` | `systemctl reload nginx` |
| Apache | `apt install apache2` | `/etc/apache2/apache2.conf` | `systemctl reload apache2` |

**Nginx commands:**
- `nginx -t`: Config test
- `nginx -s reload`: Reload
- `systemctl status nginx`: Status

**Apache commands:**
- `apache2ctl configtest`: Config test
- `a2ensite/a2dissite`: Site enable/disable
- `a2enmod/a2dismod`: Module enable/disable

---

## 🎓 Alıştırmalar

1. Nginx kur ve test et
2. Virtual host oluştur
3. SSL certificate al (Let's Encrypt)
4. Reverse proxy kur (backend app'e)
5. Load balancing yapılandır
6. Rate limiting ekle
7. Apache virtual host oluştur
8. .htaccess ile password protection
9. PHP-FPM ile Nginx entegrasyonu
10. Nginx vs Apache performans karşılaştırması

---

**Sonraki Bölüm:** [17-Database-Yonetimi.md](17-Database-Yonetimi.md) → MySQL/MariaDB, PostgreSQL, Redis, backup, replication
