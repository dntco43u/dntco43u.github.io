---
date: 2025-12-15
title: guacamole
description: guacamole 구성
tags:
  - guacamole
  - docker
categories:
  - apps
---

![image](/images/guacamole-logo.webp#center)

```mermaid
graph LR
  subgraph gvp6nx1a
  A[guacamole] <-- proxy --> C1[nginx]
  B2[ssh-servers] <-- tcp --> A
  end
  C1 <-- https --> C[client]
```

![image](/images/guacamole-1.webp)

## container 구성

### docker-compose.yml
```sh
vi /opt/guacamole/docker-compose.yml
```
```yml
services:
  guacamole:
    image: flcontainers/guacamole:latest
    container_name: guacamole
    networks:
      - dev
    ports:
      - 8080/tcp
    user: 0:0
    environment:
      - TZ=Asia/Seoul
      - EXTENSIONS=auth-totp
    volumes:
      - /opt/guacamole/config:/config:rw
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
networks:
  dev:
    external: true
```

### proxy 구성
```sh
vi /opt/nginx/config/sites-available/guacamole.conf
```
```
...
  location / {
    if ($allowed_country = no) {
      return 403;
    }
    include                 /etc/nginx/conf.d/include/proxy.conf;
    proxy_pass              http://guacamole:8080;
    proxy_buffering         off;
    proxy_request_buffering off;
  }
...
```

## Troubleshooting
{{% alert color=info %}}
> 초기 계정

guacadmin/guacadmin
{{% /alert %}}
