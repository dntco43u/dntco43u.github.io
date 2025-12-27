---
date: 2025-12-15
title: azvj6fml
description: springboot + vue.js 화면 샘플
tags:
  - oracle
  - sptingboot
  - rest-api
  - vue.js
  - docker
categories:
  - dev
---

{{% alert title="작성 중" color="primary" %}}{{% /alert %}}

![image](/images/vue-js-logo.webp#center)

## GitHub repository
[바로 가기](https://github.com/dntco43u/azvj6fml)

## container 구성

### Dockerfile
```sh
vi /opt/azvj6fml/Dockerfile
```
```dockerfile
FROM ibm-semeru-runtimes:open-17-jre
#RUN adduser --no-create-home -u 1000 dev
RUN mkdir -p /usr/share/java && \
  chown -R dev /usr/share/java
USER 1000
COPY --chown=1000:1000 build/libs/*.jar /usr/share/java/app.jar
WORKDIR /usr/share/java
EXPOSE 61314/tcp
ENTRYPOINT ["java", "-Dspring.profiles.active=prod", "-Xms2G", "-Xmx2G", "-jar", "/usr/share/java/app.jar"]
```

### docker-compose.yml
```sh
vi /opt/azvj6fml/docker-compose.yml
```
```yml
services:
  azvj6fml:
    image: e7hnr8ov/azvj6fml:open-17-jre
    container_name: azvj6fml
    networks:
      - dev
    ports:
      - 61314/tcp
    user: 1000:1000
    environment:
      - TZ=Asia/Seoul
    volumes:
      - /opt/azvj6fml/log:/usr/share/java/log:rw
    restart: unless-stopped
networks:
  dev:
    external: true
```

## host 구성

### post_build.sh [^2]
```sh
vi /opt/bdkqpr0x/config/post_build.sh
```
```sh
#!/bin/bash
# 자동 build 후 처리

source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

jenkins_bot_key="$3"
tel_chat_id="$4"
{ echo "$1 #$2"
  show_file_stat "/opt/$1/build/libs/$1-0.0.1-SNAPSHOT.jar"
  cd "/opt/$1" || exit 1
  docker build -t "e7hnr8ov/$1:open-17-jre" "/opt/$1"
  if [ -f "docker-compose.yml" ]; then
    docker compose rm -f -s
    docker compose up -d
    docker exec -i "$1" date +"%Z %F %T"
  fi
} > "$log_file"

read -ra args < <(docker images -q -f dangling=true)
docker rmi "${args[@]}"
rm -rf "/opt/$1/build"

cp "$log_file" "$msg_file"
send_tel_msg "$jenkins_bot_key" "$tel_chat_id" "$msg_file"
rm "$msg_file"
```

### logrotate
```sh
sudo vi /etc/logrotate.d/azvj6fml
```
```
/opt/azvj6fml/log/*.log {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  create 0664 dev dev
  sharedscripts
  postrotate
    docker restart azvj6fml >/dev/null 2>&1 || true
  endscript
}
```

~~## 데모 페이지~~
![image](/images/azvj6fml-1.webp)

## Troubleshooting
{{% alert color=warning %}}
> => ERROR [2/5] RUN adduser --no-create-home -u 1000 dev

ibm-semeru-runtimes:open-17-jre 이미지 업데이트로 Dockerfile build 시 계정 생성 구성 삭제
```sh
vi /opt/bdkqpr0x/Dockerfile
```
```dockerfile
#RUN adduser --no-create-home -u 1000 dev
```
{{% /alert %}}

[^2]: https://hu.gvp6nx1a.duckdns.org/infra/jenkins/#post_buildsh
