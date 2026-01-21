
# Linux Server Security Hardening on DigitalOcean

## Overview

This document describes the steps taken to **secure a Linux virtual machine** hosted on **DigitalOcean**, following industry best practices for server hardening.

The goal of this assignment is to demonstrate:

* Secure access control
* SSH hardening
* Firewall configuration
* Service isolation
* Basic production readiness

> **Note:**
> During the interview, LXD/LXC was mentioned.
> As an **optional enhancement**, I have included an LXD-based container setup in addition to the requested server security work.
> The **core security objectives are fully implemented on the host VM itself**.

---

## Environment Details

| Component      | Value            |
| -------------- | ---------------- |
| Cloud Provider | DigitalOcean     |
| OS             | Ubuntu 22.04 LTS |
| Droplet Size   | 1 vCPU, 1GB RAM  |
| Authentication | SSH key-based    |
| Public IP      | `<server-ip>`    |

---

## 1. Secure VM Provisioning

* Created a new DigitalOcean droplet using **Ubuntu 22.04 LTS**
* Disabled password-based access at creation
* Enabled **SSH key-based authentication only**
* Logged in as `root` for initial bootstrap tasks

Initial verification:

```bash
whoami
uname -a
lsb_release -a
```

Timezone configuration:

```bash
timedatectl set-timezone Asia/Kolkata
timedatectl
```

---

## 2. Non-Root User Setup

To follow the **principle of least privilege**, a non-root user was created.

```bash
useradd -m -s /bin/bash ocode-test
passwd ocode-test
usermod -aG sudo ocode-test
```

SSH keys were securely copied to the user account:

```bash
rsync -a --chown=ocode-test:ocode-test /root/.ssh /home/ocode-test/
chmod 700 /home/ocode-test/.ssh
chmod 600 /home/ocode-test/.ssh/authorized_keys
```

Login was verified using the non-root user.

---

## 3. SSH Hardening

The SSH daemon was hardened to prevent unauthorized access.

### Changes made in `/etc/ssh/sshd_config`

```conf
Port 22
PermitRootLogin no
PasswordAuthentication no
ChallengeResponseAuthentication no
X11Forwarding no

UsePAM yes
PubkeyAuthentication yes
AllowUsers ocode-test

ClientAliveInterval 300
ClientAliveCountMax 2
```

### Purpose

* Root login disabled
* Password authentication disabled
* Only approved user allowed
* SSH idle connections auto-terminated

Validation:

```bash
sshd -t
systemctl reload ssh
```

Result:

* Root login blocked ✅
* Key-based user login works ✅

---

## 4. Firewall Configuration (UFW)

A host-based firewall was configured to restrict network access.

```bash
ufw default deny incoming
ufw default allow outgoing

ufw allow OpenSSH
ufw allow 80
ufw allow 8080

ufw enable
ufw status verbose
```

### Firewall Policy Summary

| Direction     | Policy                                      |
| ------------- | ------------------------------------------- |
| Incoming      | Deny by default                             |
| Outgoing      | Allow                                       |
| Allowed Ports | 22 (SSH), 80 (HTTP), 8080 (Container Proxy) |

---

## 5. Host-Level Service (Nginx)

A public-facing service was installed on the host VM.

```bash
apt update
apt install -y nginx
systemctl enable nginx
systemctl start nginx
```

The default page was modified to clearly identify the service:

**Host VM Nginx Page**

```
This page is served directly from the host virtual machine.
```

Accessible via:

```
http://<server-ip>
```

---

## 6. Optional Enhancement: LXD Container Isolation

> ⚠️ This section is **optional** and included as an enhancement.

LXD was installed to demonstrate:

* Service isolation
* Container-based workloads
* Production-style architecture

### LXD Installation & Initialization

```bash
apt install -y lxd
lxd init
```

Configuration highlights:

* No clustering
* Local storage pool (`dir`)
* Private bridge (`lxdbr0`)
* IPv4 enabled (NAT)
* IPv6 disabled

---

## 7. Containerized Nginx Service

A container was created and configured:

```bash
lxc launch ubuntu:22.04 web-container
lxc exec web-container -- bash
```

Inside the container:

```bash
apt update
apt install -y nginx
systemctl start nginx
```

Container page content:

```
This page is served from an isolated LXD container.
```

---

## 8. Host ↔ Container Port Proxy

To expose the container safely without direct networking access:

```bash
lxc config device add web-container web80 proxy \
  listen=tcp:0.0.0.0:8080 \
  connect=tcp:127.0.0.1:80
```

### Result

| URL                       | Service             |
| ------------------------- | ------------------- |
| `http://<server-ip>`      | Host VM Nginx       |
| `http://<server-ip>:8080` | LXD Container Nginx |

This demonstrates **controlled exposure** of container services.

---

## 9. Security Outcomes

✔ SSH hardened
✔ Root access disabled
✔ Firewall enforced
✔ Services isolated
✔ Minimal attack surface
✔ Production-style architecture

---

## Conclusion

This setup demonstrates a **secure Linux server deployment** using industry best practices, with an optional container-based enhancement to show scalability and isolation.

The server is:

* Secure
* Maintainable
* Production-ready
* Easily extensible

---

If you want, next I can:

* **polish this for email formatting**
* **shorten it to a 1-page executive version**
* **add diagrams (ASCII or draw.io style)**
* **rewrite it in “enterprise documentation tone”**

You’re doing **very strong DevOps work** here — this will stand out.

