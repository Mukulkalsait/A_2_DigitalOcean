# A_2_DigitalOcean -> Linux VM + LXD + Nginx Setup Checklist

## Phase 1: DigitalOcean VM Setup
  - Droplet 1gb/1cpu
  - Name: TestDroplet 
  - Tag: UbuntuDroplet
  - ssh key
  * ssh key generation
      ```bash 
    ssh-keygen -t ed25519 -f <filename> -C "username"
    ```
    
### verification: 
  ```bash 
  uname -a 
  lsb_release -a 
  whoami
  ```


  * Local Setup for ssh config  
  > .ssh/config
  ```.ssh/config
  Host User1
    Hostname <ip>
    User root
    IdentifyFile <location/of/file>
  ```

## Phase 2: Basic config + upgrades

### User Creation:
  ```bash 
  useradd -m -s /bin/bash mukuldk
  usermod -aG sudo mukuldk
  passwd mukuldk
  ```

### Updates: 
  ```bash 
  # System Updates
  sudo apt update -y && sudo apt update -y

  # Automatic Patching for critical Security Updates
  sudo apt install unattended-upgrades 
  sudo dpkg-reconfigure --proirity=low unattended-upgrades
  ```

  * upgrade 
  >.ssh/config
  ```config
  Host User1
    Hostname <ip>
    User root
    IdentifyFile <location/of/file>

  Host mukuldk
    Hostname <ip>
    User mukuldk
    IdentifyFile <location/of/file>
  ```

## Phase 2: SSH Hardening

  * created
  > ./Config/sshd_config

  ### rsync with ssh config to update the file to server.
  ```bash 
  rsync -av ./Config/sshd_config mukuldk:~/Config/
```

  ### Backup of oringal ssh 
   ```bash  
    #as mukuldk
    mkdir ~/Config_default
    sudo cp -r /etc/ssh ~/Config_default/
    ```

  ### Copy the config 
  ```bash 
    sudo cp -r /Config/sshd_
    ```
 
  ### Verify:
   ```bash
ssh User1 => failed ❌ 
ssh mukuldk => worked. ✅
```

## Phase 3: Firewall (UFW) + Fail2ban

sudo apt install ufw fail2ban 

sudo ufw status verbose
sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow OpenSSH 

sudo ufw status verbose
sudo ufw enable 

sudo ufw status verbose

sudo apt install nginx 



## Phase 5: LXD Setup
## Phase 4: Host Nginx Setup
## Phase 6: ContainerNginx
## Phase 7: Host ↔ Container Port Proxy
## Final Verification
