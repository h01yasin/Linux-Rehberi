# 🐧 LINUX REHBERİ — BÖLÜM 20: CI/CD PIPELINES

Jenkins, GitLab CI, GitHub Actions, continuous integration, continuous deployment.

---

## CI/CD Nedir?

**CI (Continuous Integration):**
- Code commit edildiğinde otomatik build ve test
- Erken bug tespiti
- Kod kalitesi artışı

**CD (Continuous Delivery/Deployment):**
- Otomatik deployment (staging, production)
- Hızlı release cycle
- Daha az manuel işlem

**CI/CD Pipeline:**
```
Code Push → Build → Test → Deploy (Staging) → Deploy (Production)
```

---

## GitHub Actions

GitHub'da built-in CI/CD.

### Workflow Dosyası

**.github/workflows/ci.yml:**
```yaml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
```

---

### Docker Build & Push

**.github/workflows/docker.yml:**
```yaml
name: Docker Build and Push

on:
  push:
    branches: [ main ]
    tags:
      - 'v*'

env:
  REGISTRY: docker.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

### Deploy to Server

**.github/workflows/deploy.yml:**
```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/myapp
            git pull origin main
            docker compose pull
            docker compose up -d --build
            docker compose exec -T app npm run migrate
```

**GitHub Secrets:**
- Settings → Secrets → Actions → New repository secret
- `SERVER_HOST`, `SERVER_USER`, `SSH_PRIVATE_KEY`

---

### Matrix Build (Multiple Versions)

```yaml
name: Matrix Build

on: [push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        node-version: [16, 18, 20]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node ${{ matrix.node-version }} on ${{ matrix.os }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      
      - run: npm install
      - run: npm test
```

---

## GitLab CI/CD

GitLab'da built-in CI/CD.

### .gitlab-ci.yml

```yaml
image: node:18

stages:
  - build
  - test
  - deploy

variables:
  NODE_ENV: production

cache:
  paths:
    - node_modules/

before_script:
  - npm install

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

test:
  stage: test
  script:
    - npm run lint
    - npm test
  coverage: '/Coverage: \d+\.\d+%/'

deploy_staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    - rsync -avz --delete dist/ user@staging-server:/var/www/app/
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - rsync -avz --delete dist/ user@prod-server:/var/www/app/
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual  # Manual trigger
```

---

### Docker Build (GitLab)

```yaml
image: docker:latest

services:
  - docker:dind

variables:
  DOCKER_DRIVER: overlay2
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG

stages:
  - build
  - push
  - deploy

before_script:
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

build:
  stage: build
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG

deploy:
  stage: deploy
  script:
    - ssh user@server "docker pull $IMAGE_TAG && docker compose up -d"
  only:
    - main
```

---

### GitLab Runner (Self-hosted)

```bash
# Runner kurulum
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt install gitlab-runner

# Register runner
sudo gitlab-runner register

# Soru cevapları:
# - GitLab URL: https://gitlab.com/
# - Registration token: (GitLab'dan al)
# - Description: My Runner
# - Tags: docker, linux
# - Executor: docker
# - Default image: alpine:latest

# Runner başlat
sudo gitlab-runner start
```

---

## Jenkins

Self-hosted, plugin-based CI/CD.

### Kurulum

```bash
# Java kur (gerekli)
sudo apt update
sudo apt install openjdk-11-jdk

# Jenkins repo ekle
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Jenkins kur
sudo apt update
sudo apt install jenkins

# Start & enable
sudo systemctl enable --now jenkins

# Initial password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Web UI: http://localhost:8080
```

---

### Jenkinsfile (Declarative Pipeline)

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    environment {
        NODE_ENV = 'production'
        DOCKER_IMAGE = 'myapp'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/user/repo.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results/**/*.xml'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh '''
                    ssh user@staging-server '
                        cd /var/www/app &&
                        docker compose pull &&
                        docker compose up -d
                    '
                '''
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh '''
                    ssh user@prod-server '
                        cd /var/www/app &&
                        docker compose pull &&
                        docker compose up -d
                    '
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
            // Slack notification
            slackSend(color: 'good', message: "Build #${BUILD_NUMBER} succeeded")
        }
        failure {
            echo 'Pipeline failed!'
            slackSend(color: 'danger', message: "Build #${BUILD_NUMBER} failed")
        }
        always {
            cleanWs()
        }
    }
}
```

---

### Jenkins Plugins

**Gerekli Plugins:**
- Git Plugin
- Docker Pipeline
- Pipeline
- SSH Agent
- Blue Ocean (modern UI)
- NodeJS Plugin
- Slack Notification

**Plugin kurulum:**
Manage Jenkins → Manage Plugins → Available → Search & Install

---

## CI/CD Best Practices

### 1. Pipeline as Code

Jenkinsfile/workflow'ları repo'da tut.

```
project/
├── .github/
│   └── workflows/
│       └── ci.yml
├── Jenkinsfile
├── .gitlab-ci.yml
├── Dockerfile
└── docker-compose.yml
```

---

### 2. Automated Testing

```yaml
# GitHub Actions
- name: Run unit tests
  run: npm test

- name: Run integration tests
  run: npm run test:integration

- name: Run E2E tests
  run: npm run test:e2e
```

---

### 3. Multi-Environment

```yaml
# .gitlab-ci.yml
deploy_dev:
  stage: deploy
  script:
    - deploy.sh dev
  environment:
    name: development
  only:
    - develop

deploy_staging:
  stage: deploy
  script:
    - deploy.sh staging
  environment:
    name: staging
  only:
    - main

deploy_production:
  stage: deploy
  script:
    - deploy.sh production
  environment:
    name: production
  only:
    - main
  when: manual
```

---

### 4. Secrets Management

**GitHub Actions:**
```yaml
- name: Deploy
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh
```

**GitLab CI:**
```yaml
# Settings → CI/CD → Variables
deploy:
  script:
    - echo $DATABASE_URL
    - ./deploy.sh
```

**Jenkins:**
```groovy
withCredentials([string(credentialsId: 'db-password', variable: 'DB_PASS')]) {
    sh "echo $DB_PASS | docker secret create db_pass -"
}
```

---

### 5. Caching

**GitHub Actions:**
```yaml
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

**GitLab CI:**
```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/
```

---

### 6. Parallel Jobs

**GitHub Actions:**
```yaml
jobs:
  test:
    strategy:
      matrix:
        test-type: [unit, integration, e2e]
    steps:
      - run: npm run test:${{ matrix.test-type }}
```

**GitLab CI:**
```yaml
test_unit:
  stage: test
  script: npm run test:unit

test_integration:
  stage: test
  script: npm run test:integration

test_e2e:
  stage: test
  script: npm run test:e2e
```

---

## Blue-Green Deployment

Zero-downtime deployment.

```bash
# docker-compose.blue.yml
version: '3.8'
services:
  app-blue:
    image: myapp:v1.0
    container_name: app-blue
    ports:
      - "8001:3000"

# docker-compose.green.yml
version: '3.8'
services:
  app-green:
    image: myapp:v2.0
    container_name: app-green
    ports:
      - "8002:3000"
```

**Deploy script:**
```bash
#!/bin/bash

# Start green (new version)
docker compose -f docker-compose.green.yml up -d

# Health check
until curl -f http://localhost:8002/health; do
  sleep 5
done

# Switch nginx to green
sed -i 's/8001/8002/g' /etc/nginx/sites-available/myapp
nginx -s reload

# Stop blue (old version)
docker compose -f docker-compose.blue.yml down
```

---

## Canary Deployment

Gradual rollout (10% → 50% → 100%).

**Nginx config:**
```nginx
upstream backend {
    server app-v1:3000 weight=90;  # Old version
    server app-v2:3000 weight=10;  # New version (canary)
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

---

## Full CI/CD Example

### Node.js App + Docker + GitHub Actions

**Project structure:**
```
myapp/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── src/
├── tests/
├── Dockerfile
├── docker-compose.yml
├── package.json
└── .dockerignore
```

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    image: ghcr.io/username/myapp:latest
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    restart: always
    depends_on:
      - db
  
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    restart: always

volumes:
  db_data:
```

**.github/workflows/deploy.yml:**
```yaml
name: Build and Deploy

on:
  push:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
  
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/myapp
            echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            docker compose pull
            docker compose up -d
            docker compose exec -T app npm run migrate
            docker system prune -f
```

---

## Monitoring & Alerts

### Slack Notification (GitHub Actions)

```yaml
- name: Slack notification
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Deployment completed!'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
  if: always()
```

### Email Notification (Jenkins)

```groovy
post {
    failure {
        emailext(
            subject: "Build ${BUILD_NUMBER} failed",
            body: "Check ${BUILD_URL} for details",
            to: "team@example.com"
        )
    }
}
```

---

## 🎯 Özet

| Platform | Config File | Trigger | Use Case |
|----------|-------------|---------|----------|
| GitHub Actions | `.github/workflows/*.yml` | Push, PR, Schedule | Public repos, simple workflows |
| GitLab CI | `.gitlab-ci.yml` | Push, MR, Schedule | GitLab repos, all-in-one platform |
| Jenkins | `Jenkinsfile` | SCM poll, Webhook | Enterprise, complex pipelines |

**CI/CD Flow:**
```
Code Commit → Build → Test → Deploy (Staging) → Manual Approval → Deploy (Production)
```

---

## 🎓 Alıştırmalar

1. GitHub Actions ile basit CI pipeline oluştur (build + test)
2. Docker image build et ve Docker Hub'a push et
3. GitLab CI ile multi-stage pipeline yaz
4. Jenkins kur, freestyle job oluştur
5. Jenkins Pipeline (Jenkinsfile) yaz
6. SSH ile otomatik deployment kur
7. Blue-green deployment implement et
8. Secrets management (GitHub Secrets/GitLab Variables)
9. Slack notification ekle
10. Full stack app için complete CI/CD pipeline (dev → staging → prod)

---

## 🎉 Linux Rehberi Tamamlandı!

**20 Bölüm Tamamlandı:**
1. ✅ Giriş ve Dağıtımlar
2. ✅ Temel Komutlar
3. ✅ Dosya Sistemi ve İzinler
4. ✅ Paket Yönetimi
5. ✅ Kullanıcı ve Grup Yönetimi
6. ✅ Süreç Yönetimi
7. ✅ Ağ Yönetimi
8. ✅ Shell Scripting
9. ✅ Text İşleme (grep, sed, awk)
10. ✅ Servis ve Init Sistemleri (systemd)
11. ✅ Güvenlik (SSH, SSL, SELinux)
12. ✅ Logging & Monitoring
13. ✅ Disk ve Depolama (LVM, RAID)
14. ✅ Performance Tuning
15. ✅ Backup ve Restore
16. ✅ Web Serverleri (Nginx, Apache)
17. ✅ Database Yönetimi (MySQL, PostgreSQL, Redis)
18. ✅ Ansible (Configuration Management)
19. ✅ Docker ve Konteynerler
20. ✅ CI/CD Pipelines (Jenkins, GitLab CI, GitHub Actions)

**Artık yapabilirsiniz:**
- 🚀 Production server yönetimi
- 🔧 System administration (disk, network, performance)
- 🐳 Containerization (Docker + Compose)
- 🤖 Infrastructure automation (Ansible)
- ⚙️ CI/CD pipeline kurulumu
- 📊 Monitoring & logging
- 🔐 Security hardening
- 💾 Backup & disaster recovery

**DevOps Engineer ve Linux System Administrator olmaya hazırsınız!** 🎓

---

**GitHub:** Tüm rehberi GitHub'da yayınla, topluluk ile paylaş!
**Katkı:** Issues ve PR'larla rehberi geliştir!
**İletişim:** Sorular için issue aç!

---

**Başarılar! 🐧🚀**
