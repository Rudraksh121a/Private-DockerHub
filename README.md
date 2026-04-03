# 🚀 Private Docker Registry with Azure Blob Storage + Nginx + SSL

## 📌 Overview

This project demonstrates how to set up a **self-hosted private Docker image registry** using:

* Docker Registry (`registry:2`)
* Nginx (reverse proxy)
* SSL (Let's Encrypt)
* Authentication (htpasswd)
* Azure Blob Storage (as backend storage)

---

## 🧱 Architecture

```
Docker Client → Nginx (HTTPS) → Docker Registry → Azure Blob Storage
```

---

## ⚙️ Prerequisites

* Linux server (Ubuntu recommended)
* Domain name (e.g., `hub.yourdomain.com`)
* Docker & Docker Compose installed
* Azure account (for Blob Storage)

---

## 🐳 Step 1: Install Docker

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## ☁️ Step 2: Azure Blob Storage Setup

1. Create **Storage Account**
2. Create **Container** (e.g., `registry`)
3. Copy:

   * `accountname`
   * `accountkey`

---

## 📁 Step 3: Project Setup

```bash
mkdir ~/docker-registry
cd ~/docker-registry
mkdir auth
```

---

## 🔧 Step 4: Create `config.yml`

```yaml
version: 0.1

storage:
  azure:
    accountname: <YOUR_ACCOUNT_NAME>
    accountkey: <YOUR_ACCOUNT_KEY>
    container: registry

http:
  addr: :5000
```

---

## 🔐 Step 5: Setup Authentication

```bash
sudo apt install apache2-utils -y
htpasswd -Bc auth/registry.password <username>
```

---

## 🐳 Step 6: Create `docker-compose.yml`

```yaml
version: '3'

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
    volumes:
      - ./auth:/auth
      - ./config.yml:/etc/docker/registry/config.yml
```

---

## 🌐 Step 7: Nginx Setup

```bash
sudo apt install nginx
```

Edit config:

```bash
sudo vim /etc/nginx/sites-available/registry
```

```nginx
server {
    listen 80;
    server_name hub.yourdomain.com;

    location / {
        proxy_pass http://localhost:5000;

        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_read_timeout 900;
    }
}
```

Enable:

```bash
sudo ln -s /etc/nginx/sites-available/registry /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔒 Step 8: Enable SSL

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

---

## 📦 Step 9: Run Registry

```bash
docker compose up -d
```

Test:

```bash
curl https://hub.yourdomain.com/v2/
```

Expected:

```
{}
```

---

## 📤 Step 10: Push Image

```bash
docker login hub.yourdomain.com

docker tag nginx hub.yourdomain.com/myimage
docker push hub.yourdomain.com/myimage
```


## 🙌 Author

**Rudraksh**


---
