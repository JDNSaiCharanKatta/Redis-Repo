# Redis Stack Deployment with Docker, Nginx & SSL

This project demonstrates how to deploy **Redis Stack** using **Docker**, configure **Nginx Reverse Proxy**, enable **HTTPS with Let's Encrypt SSL**, and manage **Redis ACL users** on an Ubuntu server.

---

# Prerequisites

Before starting, ensure you have:

* Ubuntu 22.04 or later
* Root or sudo access
* A registered domain name
* DNS A record pointing to your server

Example:

```text
redis.charan.click ---> Your Server Public IP
```

---

# Architecture

```text
                 Internet
                     │
               HTTPS (443)
                     │
          Nginx Reverse Proxy
                     │
         Redis Stack Web UI (8001)
                     │
          Redis Docker Container
                     │
          Persistent Docker Volume
```

---

# Step 1: Update the Server

```bash
sudo apt update -y
```

---

# Step 2: Install Docker and Docker Compose

```bash
sudo apt install docker.io docker-compose -y
```

Verify installation:

```bash
docker --version
docker-compose --version
```

---

# Step 3: Start and Enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

---

# Step 4: Create Project Directory

```bash
mkdir redis-docker
cd redis-docker
```

---

# Step 5: Create Docker Compose File

Create the file:

```bash
vim docker-compose.yml
```

Paste:

```yaml
version: "3.8"

services:
  redis:
    image: redis:latest
    container_name: redis-server
    command: ["redis-server", "/usr/local/etc/redis/redis.conf"]
    ports:
      - "6379:6379"
    volumes:
      - ./redis/conf/redis.conf:/usr/local/etc/redis/redis.conf:ro
      - ./redis/data:/data
    restart: always

  redisinsight:
    image: redis/redisinsight:latest
    container_name: redis-ui
    ports:
      - "8001:5540"  # ✅ RedisInsight runs internally on port 5540
    volumes:
      - ./redisinsight/data:/db
    restart: always

```

---

# Step 6: Start Redis Stack

```bash
docker-compose up -d
docker ps
```

---

# Step 7: Install Nginx and Certbot

```bash
sudo apt install nginx certbot python3-certbot-nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

# Step 8: Configure Nginx (HTTP)

Create the configuration file:

```bash
sudo vim /etc/nginx/sites-available/redis.charan.click
```

Add:

```nginx
server {
    listen 80;
    server_name redis.charan.click;

    location / {
        proxy_pass http://127.0.0.1:8001;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

# Step 9: Enable the Nginx Site

```bash
sudo ln -s /etc/nginx/sites-available/redis.charan.click /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

# Step 10: Generate SSL Certificate

```bash
sudo certbot --nginx -d redis.charan.click
```

---

## Step 11: Configure HTTPS

Replace the Nginx configuration with the following:

```nginx
server {
    listen 80;
    server_name redis.charan.click;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name redis.charan.click;

    ssl_certificate /etc/letsencrypt/live/redis.charan.click/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/redis.charan.click/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://127.0.0.1:8001;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

Verify the configuration:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

# Step 12: Install Redis CLI


sudo apt install redis-tools -y
redis-cli --version
<img width="762" height="181" alt="image" src="https://github.com/user-attachments/assets/9443737a-df4d-4ab5-80e7-c69a4c82ff60" />  redis cli

---

# Step 13: Connect to Redis

Using domain:

```bash
redis-cli -h redis.charan.click -p 6379
```

Using server IP:

bash
redis-cli -h <SERVER-IP> -p 6379
<img width="1553" height="666" alt="image" src="https://github.com/user-attachments/assets/39e71129-872d-431c-951f-96d8ca20e0ce" /> acl list 

# Step 14: Create Redis Users

```redis
ACL SETUSER asit on >Asit@123
ACL SETUSER Charan on >Charan@366
```

---

# Step 15: Verify Users

redis
ACL LIST
<img width="1090" height="605" alt="image" src="https://github.com/user-attachments/assets/36f97ed7-f825-4184-b031-8bd9fce03e1e" /> 

# Step 16: Login Using ACL User

```bash
redis-cli --user Charan --pass "Charan@366"
```

or

```bash
redis-cli -h redis.charan.click -p 6379 --user Charan --pass "Charan@366"
```

---

# Step 17: Authenticate

```redis
AUTH Charan Charan@366
ACL WHOAMI
```

---

# Step 18: Store and Retrieve Data

```redis
SET Name "Charan"
GET Name

SET City Hyderabad
GET City

MSET Name Charan City Hyderabad Country India
MGET Name City Country

DEL Name
KEYS *
```

---

# Step 19: Useful Redis Commands

| Command         | Description                        |
| --------------- | ---------------------------------- |
| `SET key value` | Store data                         |
| `GET key`       | Retrieve data                      |
| `DEL key`       | Delete data                        |
| `KEYS *`        | List all keys                      |
| `MSET`          | Store multiple values              |
| `MGET`          | Retrieve multiple values           |
| `INFO`          | Server information                 |
| `ACL LIST`      | List all users                     |
| `ACL WHOAMI`    | Display current authenticated user |


# Step 20: Screenshots


<img width="1908" height="996" alt="image" src="https://github.com/user-attachments/assets/1e5465b9-dd03-429a-a20a-373ae877369e" /> 
<img width="1892" height="1020" alt="image" src="https://github.com/user-attachments/assets/6289030c-4b22-4d2a-9134-ece659f6442d" />
<img width="1920" height="1011" alt="image" src="https://github.com/user-attachments/assets/7ec2a059-d079-4ff3-ab33-90e6348d1985" />
<img width="705" height="304" alt="image" src="https://github.com/user-attachments/assets/f4ebfa2c-5a49-469f-aa43-33fe64a9c263" />
<img width="1920" height="1018" alt="image" src="https://github.com/user-attachments/assets/5189127b-3855-4ebc-8d1f-c0d9e6f64e54" />



