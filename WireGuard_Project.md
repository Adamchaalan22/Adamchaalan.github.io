# WireGuard VPN Project (Project 3)
**By Adam Chaalan**  
CYB 3353 – System Administration  
Instructor: Professer Codi West

---

## Overview
For this project, I deployed a WireGuard VPN Server inside a docker container on a DigitalOcean Ubuntu Droplet. The goal was to configure a working VPN that connects my mobile phone and PC, verifies traffic routing through the server, and documents the setup thoroughly using Docker Compose.

---

##  Step 1: Create DigitalOcean Droplet
1. Signed up using referral link to get $200 credit.
2. Created a new droplet:
   - **Ubuntu 24.04 x64**
   - **Regular CPU / $6/month**
   - Enabled password login
   - Selected nearest data center
   - Skipped backups/add-ons

---

## Step 2: Install Docker on Droplet

Install Docker using official Docker docs:
```bash
sudo apt update
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg]   https://download.docker.com/linux/ubuntu   $(lsb_release -cs) stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

---

## Step 3: Deploy WireGuard with Docker Compose (In-Depth)

### Clone the Official Repository
```
git clone https://github.com/linuxserver/docker-wireguard.git
cd docker-wireguard
```

### Create Docker Compose File
```
nano docker-compose.yml
```

Paste the following into the file:
```yaml
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
      - SERVERURL=165.22.154.25
      - SERVERPORT=51820
      - PEERS=2
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.13.13.0
    volumes:
      - ./config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

- The number of peers is set to 2: one for phone, one for PC.
- This mounts configuration files under `./config`.

## Step 4: Launch the Container

Start the VPN server using Docker Compose:

```bash
sudo docker compose up -d
```

This will:
- Pull the latest WireGuard image
- Set up networking
- Start the container in detached mode

---

## View Logs and Access Peer Configs

To check that WireGuard initialized properly and to obtain your peer configuration files, run:

```bash
sudo docker logs -f wireguard
```

Look for output like this:

```
Peer 1 config file created: /config/peer1/peer1.conf
```


### Generate QR Code for Easy Phone Setup

```bash
sudo apt install qrencode -y
qrencode -t ansiutf8 < config/peer1/peer1.conf
```

---

## Step 5: Confirm It’s Running

To check that the container is active:

```bash
sudo docker ps
```




<img width="660" height="1434" alt="IMG_2702" src="https://github.com/user-attachments/assets/ff757f81-9e0a-4c37-a2b2-40de1ded7812" />

<img width="660" height="1434" alt="IMG_2703" src="https://github.com/user-attachments/assets/4bcf4ca7-fbac-420d-8f7f-c7f5f53ab47c" />

<img width="1440" height="778" alt="Screenshot 2025-11-20 at 4 09 12 PM" src="https://github.com/user-attachments/assets/6a0c6c3e-52c1-4953-8295-a57ef8f48ddd" />

<img width="1440" height="775" alt="Screenshot 2025-11-20 at 4 18 23 PM" src="https://github.com/user-attachments/assets/08a4558e-d3c2-4c36-8868-76abf631dc61" />

<img width="1440" height="776" alt="Screenshot 2025-11-20 at 4 16 26 PM" src="https://github.com/user-attachments/assets/69084396-052f-4711-bea6-d3c9c8649c53" />

<img width="1440" height="545" alt="Screenshot 2025-11-20 at 4 34 27 PM" src="https://github.com/user-attachments/assets/7c5a5f90-da81-4334-83c3-57119873e413" />

<img width="1439" height="486" alt="Screenshot 2025-11-20 at 4 37 19 PM" src="https://github.com/user-attachments/assets/a2aa57fa-a199-402d-859a-a5c92e84ee42" />

<img width="1440" height="778" alt="Screenshot 2025-11-20 at 4 15 08 PM" src="https://github.com/user-attachments/assets/1d915703-984f-41a6-80ee-18999bda9bb9" />

