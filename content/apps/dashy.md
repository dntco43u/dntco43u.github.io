---
date: 2025-12-14
title: dashy
description: dashy 구성
tags:
  - dashy
  - docker
categories:
  - apps
---

![image](/images/dashy-logo.webp#center)

```mermaid
graph LR
  subgraph cloud
  A1[ec4mrjp5] <-- https --> B[dashy]
  A2[m7jrgve9] <-- https --> B
  A3[gvp6nx1a] <-- http --> B
  end
  A4[agq3mbw2] <-- https --> B
  subgraph home-lab
  A5[sj9n7air] <-- http --> A4
  end
  B <-- https --> A6[client]
```

## container 구성

### docker-compose.yml
```sh
vi /opt/dashy/docker-compose.yml
```
```yml
services:
  dashy:
    image: lissy93/dashy:latest
    container_name: dashy
    networks:
      - dev
    ports:
      - 8080/tcp
    user: 0:0
    environment:
      - UID=1000
      - GID=1000
      - MODE_ENV=production
      - TZ=Asia/Seoul
    volumes:
      - /opt/dashy/config/conf.yml:/app/user-data/conf.yml:rw
      - /opt/dashy/item-icons:/app/user-data/item-icons:rw
    restart: unless-stopped
networks:
  dev:
    external: true
```

### 암호 구성
로그인 암호 sha256 hash 생성 후 구성에 저장
```sh
echo -n "2***************************************************************" | sha256sum
```
```
7***************************************************************
```
```sh
vi /opt/dashy/config/conf.yml
```
```
appConfig:
  auth:
    users:
      - user: dev
        hash: 7***************************************************************
        type: admin
...
```

## 데모 페이지
![image](/images/dashy-1.webp)

[바로 가기](https://da.gvp6nx1a.duckdns.org)

## Troubleshooting
{{% alert color=warning %}}
> Browserslist: caniuse-lite is outdated. Please run:
>   npx update-browserslist-db@latest

 ```sh
docker exec -it dashy npx update-browserslist-db@latest
```
{{% /alert %}}
