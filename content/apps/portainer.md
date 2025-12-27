---
date: 2025-12-14
title: portainer
description: portainer 구성
tags:
  - portainer
  - docker
categories:
  - apps
---

![image](/images/portainer-logo.webp#center)

```mermaid
graph LR
  subgraph gvp6nx1a
  A1[docker.sock] ---> B
  B[portainer] <-- proxy --> C[nginx]
  end
  C <-- https --> D[client]
```

![image](/images/portainer-1.webp)

## container 구성

### docker-compose.yml
```sh
vi /opt/portainer/docker-compose.yml
```
```yml
services:
  portainer:
    image: portainer/portainer-ce:alpine
    container_name: portainer
    networks:
      - dev
    ports:
      - 8000/tcp
      - 9443/tcp
    user: 0:0
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
      - /opt/portainer/data:/data:rw
    restart: unless-stopped
networks:
  dev:
    external: true
```
