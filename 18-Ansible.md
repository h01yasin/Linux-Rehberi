# 🐧 LINUX REHBERİ — BÖLÜM 18: ANSIBLE

Configuration Management, Infrastructure as Code, automation.

---

## Ansible Nedir?

Agentless configuration management tool. SSH üzerinden remote server'ları yönetir.

**Özellikleri:**
- ⚡ Agentless (SSH ile çalışır)
- 📝 YAML syntax (kolay öğrenilir)
- 🔄 Idempotent (aynı işi tekrar çalıştırılabilir)
- 🎯 Declarative (ne istediğini söyle, nasıl yapılacağını değil)
- 📦 Modüler (2000+ built-in module)

**Use cases:**
- Configuration management
- Application deployment
- Infrastructure provisioning
- Orchestration

---

## Kurulum

**Control Node (komutların çalıştığı makine):**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# RHEL/Rocky
sudo dnf install epel-release
sudo dnf install ansible

# Via pip
pip3 install ansible

# Version
ansible --version
```

**Managed Nodes:** SSH ile erişilebilir olmalı. Agent gerektirmez.

---

## Inventory

Yönetilecek server listesi.

### /etc/ansible/hosts

```bash
sudo nano /etc/ansible/hosts
```

```ini
# Single host
web1.example.com

# Group of hosts
[webservers]
web1.example.com
web2.example.com
192.168.1.10

[databases]
db1.example.com
db2.example.com

# Group of groups
[production:children]
webservers
databases

# Variables
[webservers:vars]
ansible_user=ubuntu
ansible_port=22
ansible_python_interpreter=/usr/bin/python3

# Range
[webservers]
web[1:5].example.com
192.168.1.[10:20]
```

### Custom Inventory

```bash
# Project-specific inventory
ansible-playbook -i inventory.ini playbook.yml
```

**inventory.ini:**
```ini
[app]
app1 ansible_host=192.168.1.10 ansible_user=ubuntu
app2 ansible_host=192.168.1.11 ansible_user=ubuntu

[db]
db1 ansible_host=192.168.1.20
```

---

## Ad-Hoc Commands

Tek satır komutlar (playbook olmadan).

```bash
# Ping all hosts
ansible all -m ping

# Ping specific group
ansible webservers -m ping

# Command module (shell command çalıştır)
ansible all -m command -a "uptime"
ansible all -a "df -h"

# Shell module (pipe, redirect destekler)
ansible all -m shell -a "ps aux | grep nginx"

# Copy file
ansible webservers -m copy -a "src=/tmp/file.txt dest=/tmp/file.txt"

# Install package
ansible webservers -m apt -a "name=nginx state=present" -b
# -b = --become (sudo)

# Restart service
ansible webservers -m service -a "name=nginx state=restarted" -b

# Create user
ansible all -m user -a "name=john state=present" -b

# File permissions
ansible all -m file -a "path=/tmp/test mode=0644" -b
```

---

## Playbooks

YAML formatında automation scripts.

### Basit Playbook

**playbook.yml:**
```yaml
---
- name: Configure web server
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Copy index.html
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
        mode: '0644'
```

**Çalıştır:**
```bash
ansible-playbook playbook.yml

# Check mode (dry-run)
ansible-playbook playbook.yml --check

# Verbose
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vvv  # Very verbose

# Specific hosts
ansible-playbook playbook.yml --limit web1.example.com
```

---

### Variables

**playbook.yml:**
```yaml
---
- name: Install packages
  hosts: all
  become: yes
  
  vars:
    packages:
      - nginx
      - git
      - curl
    web_root: /var/www/html
  
  tasks:
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
    
    - name: Create web root
      file:
        path: "{{ web_root }}"
        state: directory
        mode: '0755'
```

**vars.yml (external vars):**
```yaml
packages:
  - nginx
  - git
web_root: /var/www/html
```

```yaml
- name: Use external vars
  hosts: all
  vars_files:
    - vars.yml
  tasks:
    - name: Install {{ item }}
      apt:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
```

---

### Handlers

Task'lar değişiklik yaptığında tetiklenir.

```yaml
---
- name: Configure Nginx
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
      notify: Restart nginx
    
    - name: Copy nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx
  
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

---

### Conditionals

```yaml
---
- name: Conditional tasks
  hosts: all
  become: yes
  
  tasks:
    - name: Install Apache on Ubuntu
      apt:
        name: apache2
        state: present
      when: ansible_distribution == "Ubuntu"
    
    - name: Install Apache on RHEL
      dnf:
        name: httpd
        state: present
      when: ansible_distribution == "RedHat"
    
    - name: Only on production
      command: /usr/local/bin/production-script.sh
      when: inventory_hostname in groups['production']
```

---

### Templates (Jinja2)

Dynamic config files.

**templates/nginx.conf.j2:**
```jinja
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
    
    root {{ web_root }};
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

**playbook.yml:**
```yaml
---
- name: Configure Nginx with template
  hosts: webservers
  become: yes
  
  vars:
    nginx_port: 80
    server_name: example.com
    web_root: /var/www/html
  
  tasks:
    - name: Deploy nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Reload nginx
  
  handlers:
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
```

---

### Roles

Organize playbooks modularly.

**Role structure:**
```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    ├── files/
    │   └── index.html
    ├── vars/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    └── meta/
        └── main.yml
```

**Create role:**
```bash
ansible-galaxy init nginx
```

**roles/nginx/tasks/main.yml:**
```yaml
---
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Copy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx

- name: Start nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

**roles/nginx/handlers/main.yml:**
```yaml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

**Use role in playbook:**
```yaml
---
- name: Configure web server
  hosts: webservers
  become: yes
  
  roles:
    - nginx
```

---

### Örnek: LAMP Stack

**lamp.yml:**
```yaml
---
- name: Install LAMP Stack
  hosts: webservers
  become: yes
  
  vars:
    mysql_root_password: "SecurePassword123"
    db_name: "myapp"
    db_user: "appuser"
    db_password: "AppPassword123"
  
  tasks:
    # Apache
    - name: Install Apache
      apt:
        name: apache2
        state: present
        update_cache: yes
    
    - name: Start Apache
      service:
        name: apache2
        state: started
        enabled: yes
    
    # MySQL
    - name: Install MySQL
      apt:
        name:
          - mysql-server
          - python3-pymysql
        state: present
    
    - name: Start MySQL
      service:
        name: mysql
        state: started
        enabled: yes
    
    - name: Create database
      mysql_db:
        name: "{{ db_name }}"
        state: present
        login_unix_socket: /var/run/mysqld/mysqld.sock
    
    - name: Create database user
      mysql_user:
        name: "{{ db_user }}"
        password: "{{ db_password }}"
        priv: "{{ db_name }}.*:ALL"
        state: present
        login_unix_socket: /var/run/mysqld/mysqld.sock
    
    # PHP
    - name: Install PHP
      apt:
        name:
          - php
          - libapache2-mod-php
          - php-mysql
        state: present
      notify: Restart Apache
    
    - name: Deploy info.php
      copy:
        content: "<?php phpinfo(); ?>"
        dest: /var/www/html/info.php
  
  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
```

---

### Örnek: User Management

**users.yml:**
```yaml
---
- name: Manage users
  hosts: all
  become: yes
  
  vars:
    users:
      - name: alice
        groups: sudo
        shell: /bin/bash
      - name: bob
        groups: developers
        shell: /bin/bash
      - name: charlie
        state: absent  # Delete user
  
  tasks:
    - name: Create groups
      group:
        name: developers
        state: present
    
    - name: Create/delete users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups | default('') }}"
        shell: "{{ item.shell | default('/bin/bash') }}"
        state: "{{ item.state | default('present') }}"
      loop: "{{ users }}"
    
    - name: Add SSH keys
      authorized_key:
        user: "{{ item.name }}"
        key: "{{ lookup('file', 'files/ssh_keys/' + item.name + '.pub') }}"
        state: present
      loop: "{{ users }}"
      when: item.state | default('present') == 'present'
```

---

### Örnek: Docker Deployment

**docker.yml:**
```yaml
---
- name: Deploy Docker app
  hosts: app
  become: yes
  
  vars:
    app_name: myapp
    app_image: nginx:latest
  
  tasks:
    - name: Install Docker
      apt:
        name:
          - docker.io
          - python3-docker
        state: present
    
    - name: Start Docker
      service:
        name: docker
        state: started
        enabled: yes
    
    - name: Pull Docker image
      docker_image:
        name: "{{ app_image }}"
        source: pull
    
    - name: Run container
      docker_container:
        name: "{{ app_name }}"
        image: "{{ app_image }}"
        state: started
        restart_policy: always
        ports:
          - "80:80"
        volumes:
          - /var/www/html:/usr/share/nginx/html
```

---

### Örnek: Firewall Config

**firewall.yml:**
```yaml
---
- name: Configure firewall
  hosts: all
  become: yes
  
  vars:
    allowed_ports:
      - 22
      - 80
      - 443
  
  tasks:
    - name: Install ufw
      apt:
        name: ufw
        state: present
    
    - name: Allow SSH
      ufw:
        rule: allow
        port: '22'
        proto: tcp
    
    - name: Allow ports
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop: "{{ allowed_ports }}"
    
    - name: Enable ufw
      ufw:
        state: enabled
```

---

## Ansible Vault

Encrypt sensitive data (passwords, keys).

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars.yml

# Decrypt file
ansible-vault decrypt vars.yml

# View encrypted file
ansible-vault view secrets.yml

# Change password
ansible-vault rekey secrets.yml
```

**secrets.yml:**
```yaml
---
mysql_root_password: "SecurePassword123"
api_token: "abc123xyz789"
```

**Use in playbook:**
```bash
# With password prompt
ansible-playbook playbook.yml --ask-vault-pass

# With password file
echo "my_vault_password" > .vault_pass
ansible-playbook playbook.yml --vault-password-file .vault_pass
```

```yaml
---
- name: Use vault secrets
  hosts: databases
  become: yes
  vars_files:
    - secrets.yml
  
  tasks:
    - name: Set MySQL root password
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock
```

---

## Ansible Galaxy

Public playbook/role repository.

```bash
# Search roles
ansible-galaxy search nginx

# Install role
ansible-galaxy install geerlingguy.nginx

# Install from requirements
# requirements.yml:
# - src: geerlingguy.nginx
# - src: geerlingguy.mysql

ansible-galaxy install -r requirements.yml

# List installed roles
ansible-galaxy list

# Remove role
ansible-galaxy remove geerlingguy.nginx
```

**Use Galaxy role:**
```yaml
---
- name: Use Galaxy role
  hosts: webservers
  become: yes
  
  roles:
    - geerlingguy.nginx
```

---

## Best Practices

### 1. Project Structure

```
ansible-project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts
│   │   └── group_vars/
│   └── staging/
│       ├── hosts
│       └── group_vars/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── mysql/
├── group_vars/
│   ├── all.yml
│   └── webservers.yml
├── host_vars/
│   └── web1.yml
└── files/
```

### 2. ansible.cfg

```ini
[defaults]
inventory = inventory/production/hosts
remote_user = ubuntu
private_key_file = ~/.ssh/id_rsa
host_key_checking = False
retry_files_enabled = False
roles_path = roles

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

### 3. group_vars/all.yml

```yaml
---
# Common variables for all hosts
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org

timezone: "Europe/Istanbul"

admin_users:
  - alice
  - bob
```

### 4. Naming Conventions

```yaml
# Good
- name: Install nginx package
- name: Copy nginx config
- name: Restart nginx service

# Bad
- name: task 1
- name: install
```

### 5. Idempotency

```yaml
# Good (idempotent)
- name: Ensure nginx is installed
  apt:
    name: nginx
    state: present

# Bad (not idempotent)
- name: Install nginx
  shell: apt-get install nginx
```

### 6. Error Handling

```yaml
- name: Task that might fail
  command: /usr/bin/some-command
  ignore_errors: yes

- name: Task with retry
  command: /usr/bin/flaky-command
  retries: 3
  delay: 5
  register: result
  until: result.rc == 0

- name: Task with block
  block:
    - name: Try this
      command: /usr/bin/risky-command
  rescue:
    - name: Do this if failed
      debug:
        msg: "Command failed, doing cleanup"
  always:
    - name: Always do this
      debug:
        msg: "This always runs"
```

---

## 🎯 Özet

| Component | Purpose | Example |
|-----------|---------|---------|
| Inventory | Server listesi | `/etc/ansible/hosts` |
| Playbook | Automation script | `playbook.yml` |
| Module | Görev tipleri (apt, copy, service) | `apt: name=nginx` |
| Role | Organize playbooks | `roles/nginx/` |
| Vault | Encrypt secrets | `ansible-vault create secrets.yml` |
| Galaxy | Public roles | `ansible-galaxy install geerlingguy.nginx` |

**Commands:**
```bash
ansible all -m ping                    # Test connectivity
ansible-playbook playbook.yml          # Run playbook
ansible-playbook playbook.yml --check  # Dry-run
ansible-vault create secrets.yml       # Encrypt file
ansible-galaxy install role-name       # Install role
```

---

## 🎓 Alıştırmalar

1. Inventory oluştur (3 server)
2. Ad-hoc command ile uptime al
3. Nginx kurulum playbook yaz
4. Template kullan (nginx.conf.j2)
5. Handler ekle (config değişince restart)
6. Role oluştur (nginx role)
7. LAMP stack kurulum playbook
8. User management playbook
9. Ansible Vault ile secrets yönet
10. Multi-tier app deployment (web + db + load balancer)

---

**Sonraki Bölüm:** [19-Docker-Konteyner.md](19-Docker-Konteyner.md) → Docker, containers, Docker Compose
