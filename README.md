# A_2_DigitalOcean → Linux VM + LXD + Nginx Setup Checklist
This document is a **step-by-step execution checklist** used for video demonstration and verification.

---

## Phase 1: DigitalOcean VM Setup

* Droplet: **1GB RAM / 1 vCPU**
* Name: `TestDroplet`
* Tag: `UbuntuDroplet`
* OS: Ubuntu LTS
* Authentication: **SSH key only**

### SSH key generation (local)

```bash
ssh-keygen -t ed25519 -f <filename> -C "username"
```

### Verification on server

```bash
uname -a
lsb_release -a
whoami
```

### Local SSH config

```sshconfig
Host User1
  Hostname <SERVER_IP>
  User root
  IdentityFile <PATH_TO_KEY>
```

---

## Phase 2: Basic Configuration + Upgrades

### User creation

```bash
useradd -m -s /bin/bash mukuldk
usermod -aG sudo mukuldk
passwd mukuldk
```

### System updates & security patches

```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### Updated SSH config (local)

```sshconfig
Host User1
  Hostname <SERVER_IP>
  User root
  IdentityFile <PATH_TO_KEY>

Host mukuldk
  Hostname <SERVER_IP>
  User mukuldk
  IdentityFile <PATH_TO_KEY>
```

---

## Phase 3: SSH Hardening

### Custom sshd config

* File created locally: `./Config/sshd_config`

### Sync config to server

```bash
rsync -av ./Config/sshd_config mukuldk:~/Config/
```

### Backup original SSH config (on server)

```bash
mkdir ~/Config_default
sudo cp -r /etc/ssh ~/Config_default/
```

### Apply new SSH configuration

```bash
sudo cp ~/Config/sshd_config /etc/ssh/sshd_config
sudo systemctl restart ssh
```

### Verification

```bash
ssh User1      # blocked
ssh mukuldk    # allowed
```

---

## Phase 4: Firewall (UFW) + Fail2Ban

### Install packages

```bash
sudo apt install ufw fail2ban nginx -y
```

### UFW configuration

```bash
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow OpenSSH
sudo ufw allow "Nginx Full"
sudo ufw enable
```

### Verify firewall

```bash
sudo ufw status verbose
```

### Fail2Ban verification
 
  * fiil2ban config 
  > ./Config/jail.local

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

---

## Phase 5: Host Nginx Setup

### Modify default page

```bash
sudo vim /var/www/html/index.nginx-debian.html
```

**Content:**

```
This page is served from the HOST virtual machine
```

### Verify

```bash
curl http://localhost
curl http://<SERVER_IP>
```

---

## Phase 6: LXD Setup

### Install LXD

```bash
sudo snap install lxd
sudo usermod -aG lxd mukuldk
newgrp lxd
```

### Initialize LXD

```bash
sudo lxd init
```

**Choices used:**

* Clustering: no
* Storage: dir
* Bridge: lxdbr0
* IPv4: auto
* IPv6: no
* Remote access: no

### Launch container

```bash
sudo lxc launch ubuntu:22.04 cont1
```

---

## Phase 7: Container Nginx Setup

### Enter container

```bash
sudo lxc exec cont1 -- bash
```



### setup firewall for container:
```bash
sudo ufw allow in on lxdbr0
sudo ufw route allow in on lxdbr0
sudo ufw route allow out on lxdbr0
```


### Install nginx

```bash

apt update && apt install nginx -y
```

### Modify container nginx page

```bash
vim /var/www/html/index.nginx-debian.html
```


### Verify inside container

```bash
curl http://localhost
```

---

## Phase 8: Host ↔ Container Port Proxy

Expose container nginx on **host port 8080**:

```bash
sudo lxc config device add cont1 nginx-service proxy \
  listen=tcp:0.0.0.0:8080 \
  connect=tcp:127.0.0.1:80
```

---

## Final Verification

| URL                       | Expected Result          |
| ------------------------- | ------------------------ |
| `http://<SERVER_IP>`      | Host Nginx page          |
| `http://<SERVER_IP>:8080` | LXD Container Nginx page |

---

## Notes

* Host uses **native port 80**
* Container nginx is isolated and exposed via LXD proxy
* Demonstrates security, isolation, and production-style setup

---

✅ **Task Completed Successfully**
