# Docker + Docker Compose Setup on Ubuntu (Adam Chaalan)

## Overview
For this assignment, I installed Docker and Docker Compose on my Ubuntu VM and deployed the [Uptime Kuma](https://github.com/louislam/uptime-kuma) monitoring app using Docker Compose. This documentation details every step I took, including the required commands, configurations, and common errors. All work was done on my existing Ubuntu VM running on UTM for macOS.

I also referenced Docker's official installation instructions for Ubuntu found here: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/) to help guide my steps, which matched my Ubuntu environment. Many of the foundational Docker concepts and post-install commands are the same.

---

## Step 1: Install Docker

I followed the official Docker installation guide for Ubuntu here: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

### Full Command Sequence
```bash
# 1. Update existing packages
sudo apt-get update

# 2. Install required dependencies
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# 3. Add Docker’s official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4. Set up the stable repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Update apt again
sudo apt-get update

# 6. Install Docker Engine, CLI, and Compose plugin
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 7. Test installation
sudo docker run hello-world
```

Confirm installation:
```bash
docker --version
docker info
```

Confirm installation:
```bash
docker --version
docker info
```

---

## Step 2: Add User to Docker Group
```bash
sudo usermod -aG docker adam
newgrp docker
```


---

## tep 3: Install Docker Compose (via `docker-compose-plugin`)
```bash
sudo pacman -S docker-compose
```

Check version:
```bash
docker compose version
```

---

## Step 4: Choose and Set Up the App – **Uptime Kuma**
I chose Uptime Kuma because it’s a simple and useful self-hosted monitoring tool, good for testing uptime and response time of apps, APIs, and websites.

### Create Project Directory
```bash
mkdir -p ~/docker/uptime-kuma && cd ~/docker/uptime-kuma
```
nano docker-compose.yml

### Create `docker-compose.yml`
```yaml
version: '3.8'
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: always
    volumes:
      - ./uptime-kuma-data:/app/data
    ports:
      - "3001:3001"
```

Save as `docker-compose.yml` in your `uptime-kuma` directory.

---

## Step 5: Launch the App
```bash
sudo docker compose up -d
```

Check container status:
```bash
docker ps
```

---

## Step 6: Access the App
Open a browser on your host system and visit:
```
http://localhost:3001
```


---

## Step 7: Stop or Restart App
```bash
# Stop
docker compose down

# Restart
docker compose up -d
```

---

## Files Created
- `~/docker/uptime-kuma/docker-compose.yml`
- Persistent volume folder: `uptime-kuma-data`

---

## Summary
This assignment helped me gain hands-on experience with Docker and Compose while learning how to containerize and serve a full monitoring app. It also tested my ability to troubleshoot and document the process clearly.

**— Adam Chaalan**

