---
date: 2025-12-09
title: teldrive
description: teldrive 구성
tags:
  - teldrive
  - docker
categories:
  - apps
---

![image](/images/teldrive-logo.webp#center)

```mermaid
graph LR
  subgraph gvp6nx1a
  B[teldrive] <-- proxy --> C[nginx]
  end
  C <-- https --> D[client]
```

![image](/images/teldrive-1.webp)

## container 구성

### .env
```sh
vi /opt/teldrive/.env
```
```ini
POSTGRES_USER=teldrive
POSTGRES_PASSWORD=X***************************************************************
```

### docker-compose.yml
```sh
vi /opt/teldrive/docker-compose.yml
```
```yml
services:
  teldrive:
    image: ghcr.io/tgdrive/teldrive:latest
    container_name: teldrive
    networks:
      - dev
    ports:
      - 59314:8080/tcp
    user: 1000:1000
    environment:
      - TZ=Asia/Seoul
    volumes:
      - /opt/teldrive/config/config.toml:/config.toml:rw
      - /opt/teldrive/config/storage.db:/storage.db:rw
    restart: unless-stopped
    depends_on:
      - teldrive_db
  teldrive_db:
    image: groonga/pgroonga:latest-alpine-17
    container_name: teldrive_db
    networks:
      - dev
    user: 1000:1000
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=$POSTGRES_USER
      - POSTGRES_PASSWORD=$POSTGRES_PASSWORD
      - TZ=Asia/Seoul
    volumes:
      - /opt/teldrive/data:/var/lib/postgresql/data
    restart: unless-stopped
networks:
  dev:
    external: true
```

### config.toml
```sh
vi /opt/teldrive/config/config.toml
```
```toml
[db]
data-source = "postgres://teldrive:X***************************************************************@teldrive_db/postgres"

[jwt]
secret = "7***************************************************************"

[tg]
app-id = 2*******
app-hash = "f*******************************"
rate = 50 #ban 되지 않기 위한 속도 제한
rate-burst = 20
rate-limit = true

[tg.stream]
multi-threads = 6
stream-buffers = 20

[tg.uploads]
encryption-key = "b***************************************************************"
```

### db 초기화
```sh
sudo rm -rf /opt/teldrive/data/*
```

### proxy 구성
```sh
vi /opt/nginx/config/conf.d/include/proxy.conf
```
```conf
proxy_http_version 1.1;

# Proxy SSL
proxy_ssl_server_name on;

# Proxy headers
proxy_set_header Host                 $host;
proxy_set_header Upgrade              $http_upgrade;
proxy_set_header Connection           $connection_upgrade;
proxy_set_header X-Real-IP            $remote_addr;
proxy_set_header Forwarded            $proxy_add_forwarded;
proxy_set_header X-Forwarded-For      $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto    $scheme;
proxy_set_header X-Forwarded-Protocol $scheme;
proxy_set_header X-Forwarded-Host     $host;
proxy_set_header X-Forwarded-Port     $server_port;
proxy_set_header Connection           "";

# Proxy timeouts
proxy_connect_timeout 180s;
proxy_send_timeout    180s;
proxy_read_timeout    180s;
...
```
```sh
vi /opt/nginx/config/sites-available/teldrive.conf
```
```conf
...
  location / {
    if ($allowed_country = no) {
      return 403;
    }
    include    /etc/nginx/conf.d/include/proxy.conf;
    # webdav 공통
    proxy_buffering         off;
    proxy_request_buffering off;
    client_max_body_size    0;
    keepalive_timeout       180s;
    proxy_pass http://teldrive:8080;
  }
...
```

## Troubleshooting
{{% alert color=warning %}}
> 봇 추가 시 속도 하락 (2025-10)

봇 삭제 시 속도 정상화
{{% /alert %}}
