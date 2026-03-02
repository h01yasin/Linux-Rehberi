# 🐧 LINUX REHBERİ — BÖLÜM 19: DOCKER VE KONTEYNERLER

Docker, containers, images, Docker Compose, orchestration.

---

## Docker Nedir?

Container platformu. Uygulamaları izole ortamlarda çalıştırır.

**Container vs VM:**
- Container: OS-level virtualization, lightweight, fast startup
- VM: Hardware-level virtualization, heavyweight, slow startup

**Avantajları:**
- 🚀 Hızlı deployment
- 📦 Taşınabilir (dev = prod)
- ⚡ Hafif (VM'e göre)
- 🔄 Ölçeklenebilir
- 🎯 Mikro servis mimarisi

---

## Kurulum

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# RHEL/Rocky
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io

# Start & enable
sudo systemctl enable --now docker

# User'ı docker grubuna ekle (sudo gerektirmemek için)
sudo usermod -aG docker $USER
newgrp docker

# Test
docker --version
docker run hello-world
```

---

## Docker Images

Read-only template. Container'lar image'dan oluşturulur.

```bash
# Image ara
docker search nginx

# Image indir
docker pull nginx
docker pull nginx:1.23  # Specific version

# Image listesi
docker images
docker image ls

# Image sil
docker rmi nginx
docker image rm nginx

# Kullanılmayan image'ları temizle
docker image prune

# Image detayları
docker inspect nginx

# Image history
docker history nginx
```

---

## Docker Containers

Running instance of an image.

```bash
# Container çalıştır
docker run nginx

# Background (detached)
docker run -d nginx

# İsim ver
docker run -d --name mynginx nginx

# Port mapping
docker run -d -p 8080:80 nginx
# Host:8080 -> Container:80

# Environment variable
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Volume mount
docker run -d -v /host/path:/container/path nginx

# Interactive shell
docker run -it ubuntu bash
docker run -it --rm ubuntu bash  # --rm: exit'te container'ı sil

# Container listesi
docker ps           # Running containers
docker ps -a        # All containers

# Container durdur
docker stop mynginx
docker stop $(docker ps -q)  # Tümünü durdur

# Container başlat
docker start mynginx

# Container restart
docker restart mynginx

# Container sil
docker rm mynginx
docker rm -f mynginx  # Force (çalışıyorsa)

# Container log
docker logs mynginx
docker logs -f mynginx  # Follow

# Container içine gir
docker exec -it mynginx bash

# Container stats
docker stats
docker stats mynginx

# Container detayları
docker inspect mynginx
```

---

## Dockerfile

Image create etmek için recipe.

### Basit Dockerfile

**Dockerfile:**
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```bash
# Build
docker build -t myimage .

# Tag
docker build -t myimage:v1.0 .

# Run
docker run -d -p 8080:80 myimage
```

---

### Node.js App

**app.js:**
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello from Docker!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

**Dockerfile:**
```dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

```bash
docker build -t myapp .
docker run -d -p 3000:3000 myapp
```

---

### Multi-stage Build

Küçük production image için.

**Dockerfile:**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-slim

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

### Dockerfile Best Practices

```dockerfile
# Use official base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy only requirements first (layer caching)
COPY package*.json ./
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Change ownership
RUN chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

# Start command
CMD ["node", "server.js"]
```

**.dockerignore:**
```
node_modules
npm-debug.log
Dockerfile
.git
.gitignore
README.md
.env
```

---

## Docker Volumes

Persistent data storage.

```bash
# Named volume oluştur
docker volume create mydata

# Volume listesi
docker volume ls

# Volume detayları
docker volume inspect mydata

# Volume kullan
docker run -d -v mydata:/data nginx

# Bind mount (host directory)
docker run -d -v /host/path:/container/path nginx
docker run -d -v $(pwd)/html:/usr/share/nginx/html nginx

# Read-only mount
docker run -d -v /host/path:/container/path:ro nginx

# Volume sil
docker volume rm mydata

# Kullanılmayan volume'ları temizle
docker volume prune
```

---

## Docker Networks

Container'lar arası network.

```bash
# Network oluştur
docker network create mynet

# Network listesi
docker network ls

# Network detayları
docker network inspect mynet

# Container'ı network'e bağla
docker run -d --name web --network mynet nginx

# İki container'ı aynı network'te çalıştır
docker network create myapp-network
docker run -d --name db --network myapp-network mysql
docker run -d --name app --network myapp-network myapp

# Network'ten çıkar
docker network disconnect mynet web

# Network sil
docker network rm mynet
```

**Network types:**
- `bridge`: Default, isolated network
- `host`: Container host network'ü kullanır
- `none`: No network

---

## Docker Compose

Multi-container uygulamalar için.

### Kurulum

```bash
# Docker Compose V2 (plugin)
sudo apt install docker-compose-plugin

# Test
docker compose version
```

---

### docker-compose.yml

**WordPress + MySQL:**
```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    networks:
      - wp-network
  
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - db
    networks:
      - wp-network

volumes:
  db_data:
  wp_data:

networks:
  wp-network:
```

```bash
# Start
docker compose up -d

# Stop
docker compose down

# Logs
docker compose logs
docker compose logs -f wordpress

# Restart
docker compose restart

# Rebuild
docker compose build
docker compose up -d --build

# Scale service
docker compose up -d --scale wordpress=3
```

---

### Custom App

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Backend API
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - DB_USER=apiuser
      - DB_PASSWORD=apipassword
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis
    networks:
      - app-network
    volumes:
      - ./api:/app
      - /app/node_modules
    restart: always
  
  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - api
    networks:
      - app-network
    restart: always
  
  # Database
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=apiuser
      - POSTGRES_PASSWORD=apipassword
      - POSTGRES_DB=myapp
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - app-network
    restart: always
  
  # Cache
  redis:
    image: redis:7-alpine
    networks:
      - app-network
    restart: always
  
  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    depends_on:
      - frontend
      - api
    networks:
      - app-network
    restart: always

volumes:
  db_data:

networks:
  app-network:
    driver: bridge
```

---

## Docker Registry

Private image repository.

### Docker Hub

```bash
# Login
docker login

# Tag image
docker tag myimage username/myimage:v1.0

# Push
docker push username/myimage:v1.0

# Pull
docker pull username/myimage:v1.0
```

---

### Private Registry

```bash
# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag for local registry
docker tag myimage localhost:5000/myimage

# Push to local registry
docker push localhost:5000/myimage

# Pull from local registry
docker pull localhost:5000/myimage
```

---

## Dockerfile Instructions

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM node:18` |
| `RUN` | Execute command (build time) | `RUN apt-get update` |
| `CMD` | Default command (runtime) | `CMD ["node", "app.js"]` |
| `ENTRYPOINT` | Main executable | `ENTRYPOINT ["python"]` |
| `COPY` | Copy files | `COPY . /app` |
| `ADD` | Copy + extract | `ADD archive.tar /app` |
| `WORKDIR` | Set working dir | `WORKDIR /app` |
| `ENV` | Environment variable | `ENV NODE_ENV=production` |
| `EXPOSE` | Declare port | `EXPOSE 3000` |
| `VOLUME` | Mount point | `VOLUME /data` |
| `USER` | Set user | `USER nodejs` |
| `ARG` | Build argument | `ARG VERSION=1.0` |
| `LABEL` | Metadata | `LABEL maintainer="you@example.com"` |
| `HEALTHCHECK` | Health check | `HEALTHCHECK CMD curl -f http://localhost/ || exit 1` |

---

## Docker Security

### 1. Non-root User

```dockerfile
FROM node:18

RUN groupadd -r nodejs && useradd -r -g nodejs nodejs

USER nodejs

CMD ["node", "app.js"]
```

---

### 2. Read-only Filesystem

```bash
docker run -d --read-only --tmpfs /tmp nginx
```

---

### 3. Resource Limits

```bash
# Memory limit
docker run -d -m 512m nginx

# CPU limit
docker run -d --cpus="1.5" nginx

# docker-compose.yml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

---

### 4. Security Scanning

```bash
# Docker Scout (built-in)
docker scout cves nginx

# Trivy
docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image nginx
```

---

### 5. Secrets Management

```bash
# Docker secrets (Swarm mode)
echo "my_secret_password" | docker secret create db_password -

# docker-compose.yml (with secrets)
version: '3.8'

services:
  db:
    image: postgres
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    external: true
```

---

## Production Best Practices

### 1. Health Checks

**Dockerfile:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1
```

**docker-compose.yml:**
```yaml
services:
  app:
    image: myapp
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
```

---

### 2. Logging

```bash
# View logs
docker logs myapp

# json-file driver (default)
docker run -d --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 nginx

# Send logs to syslog
docker run -d --log-driver syslog --log-opt syslog-address=tcp://192.168.1.10:514 nginx
```

**docker-compose.yml:**
```yaml
services:
  app:
    image: myapp
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

### 3. Restart Policies

```bash
# Always restart
docker run -d --restart=always nginx

# Unless stopped
docker run -d --restart=unless-stopped nginx

# On failure
docker run -d --restart=on-failure:3 nginx
```

---

### 4. Environment Variables

**.env:**
```bash
DB_HOST=db
DB_USER=myuser
DB_PASSWORD=mypassword
NODE_ENV=production
```

**docker-compose.yml:**
```yaml
services:
  app:
    image: myapp
    env_file:
      - .env
```

---

## Troubleshooting

```bash
# Container logs
docker logs -f --tail 100 myapp

# Inspect container
docker inspect myapp

# Container processes
docker top myapp

# Resource usage
docker stats myapp

# Network connections
docker network inspect bridge

# Execute command in running container
docker exec myapp ls -la /app

# Copy files from container
docker cp myapp:/app/logs/error.log ./

# Copy files to container
docker cp ./config.json myapp:/app/

# Container changes
docker diff myapp

# Export container filesystem
docker export myapp > myapp.tar

# Import image
docker import myapp.tar myapp:backup
```

---

## 🎯 Özet

| Command | Purpose |
|---------|---------|
| `docker pull nginx` | Image indir |
| `docker run -d -p 80:80 nginx` | Container çalıştır |
| `docker ps` | Running containers |
| `docker logs -f myapp` | Container logs |
| `docker exec -it myapp bash` | Container'a gir |
| `docker compose up -d` | Compose stack başlat |
| `docker compose down` | Compose stack durdur |
| `docker build -t myimage .` | Image build |
| `docker push user/image` | Image push |

---

## 🎓 Alıştırmalar

1. Docker kur ve hello-world çalıştır
2. Nginx image indir, container çalıştır
3. Custom Dockerfile yaz (Node.js app)
4. Multi-stage build ile küçük image oluştur
5. Volume kullan (persistent data)
6. Docker network oluştur, iki container'ı bağla
7. Docker Compose ile WordPress + MySQL kur
8. 3-tier app deploy et (frontend + backend + db)
9. Health check ekle
10. Private registry kur, image push/pull yap

---

**Sonraki Bölüm:** [20-CI-CD-Pipelines.md](20-CI-CD-Pipelines.md) → Jenkins, GitLab CI, GitHub Actions, DevOps automation
