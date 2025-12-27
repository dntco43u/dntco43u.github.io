---
date: 2025-12-15
title: pgsql-exporter
description: pgsql-exporter 구성
tags:
  - pgsql-exporter
  - docker
categories:
  - apps
---

![image](/images/postgresql-logo.webp#center)
```mermaid
graph TB
  subgraph gvp6nx1a
  A2[postgres] -- tcp --> A4[postgres-exporter]
  A4 -- http --> A3[prometheus]
  end
```

## container 구성

### .env
```sh
vi /opt/pgsql-exporter/.env
```
```ini
DATA_SOURCE_NAME=postgresql://postgres:a***************************************************************@host.docker.internal:5432/postgres?sslmode=disable
```

### docker-compose.yml
network bridge 구성이어도 host를 허용하도록 구성
```sh
vi /opt/pgsql-exporter/docker-compose.yml
```
```yml
services:
  pgsql-exporter:
    image: quay.io/prometheuscommunity/postgres-exporter:latest
    container_name: pgsql-exporter
    networks:
      - dev
    ports:
      - 9187/tcp
    extra_hosts:
      - "host.docker.internal:host-gateway"
    user: 1000:1000
    environment:
      - DATA_SOURCE_NAME=$DATA_SOURCE_NAME
    volumes:
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
      - /opt/pgsql-exporter/config/postgres_exporter.yml:/postgres_exporter.yml:rw
    restart: unless-stopped
networks:
  dev:
    external: true
```
