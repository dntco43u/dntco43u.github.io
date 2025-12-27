---
date: 2025-12-10
title: bdkqpr0x
description: springboot 배치 샘플
tags:
  - oracle
  - postgresql
  - mybatis
  - sptingboot
  - docker
categories:
  - dev
---

![image](/images/spring-boot-logo.webp#center)

## GitHub repository
[바로 가기](https://github.com/dntco43u/bdkqpr0x)

## container 구성

### Dockerfile
```sh
vi /opt/bdkqpr0x/Dockerfile
```
```dockerfile
FROM ibm-semeru-runtimes:open-17-jre
#RUN adduser --no-create-home -u 1000 dev
RUN mkdir -p /usr/share/java && \
  chown -R 1000 /usr/share/java
USER 1000
COPY --chown=1000:1000 build/libs/*.jar /usr/share/java/app.jar
WORKDIR /usr/share/java
```

## host 구성

### bdkqpr0x.sh [^1]
```sh
vi /home/dev/.local/bin/bdkqpr0x.sh
```
```sh
#!/bin/bash
# bdkqpr0x

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

docker run \
  -i --rm --name=bdkqpr0x --network=dev \
  --add-host=host.docker.internal:host-gateway --user=1000:1000 \
  -e TZ=Asia/Seoul \
  -v /opt/bdkqpr0x/log:/usr/share/java/log:rw \
  -v /opt/bdkqpr0x/data:/usr/share/java/data:rw \
  e7hnr8ov/bdkqpr0x:open-17-jre \
  java -Dspring.profiles.active=prod -Xms2G -Xmx2G \
  -jar /usr/share/java/app.jar \
  --job.name="$1" chunkSize="$2" requestDate="$3" > "$log_file"

echo "--job.name=$1 chunkSize=$2 requestDate=$3" > "$msg_file"
tail -n3 "$log_file" | head -n1 >> "$msg_file" #show count
send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
rm "$msg_file"
```

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
sudo vi /etc/logrotate.d/bdkqpr0x
```
```
/opt/bdkqpr0x/log/*.log {
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
    docker restart promtail >/dev/null 2>&1 || true
  endscript
}
```

## Job
| Jobs | File              | Remarks                                    |
|:-----|:------------------|:-------------------------------------------|
| bd01 | us-500.csv        | pgsql -> Oracle, pgsql -> File -> Oracle   |
| bd02 | us-500.csv        | Oracle -> pgsql, Oracle -> File -> pgsql   |
| bd03 | music_tag.csv     | Oracle -> Oracle, Oracle -> File -> Oracle |
| bd04 | system_events.csv | pgsql -> Oracle, pgsql -> File -> Oracle   |

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

{{% alert color=warning %}}
> Oracle 부적합한 열유형

```sh
vi $PROJECT_PATH/src/main/resources/mybatis-config.xml
```
```xml
...
<setting name="jdbcTypeForNull" value="NULL" />
...
```
mybaris oracle에서 paramter값 null 처리 구성
{{% /alert %}}

[^1]: https://github.com/dntco43u/s6h7k8rv/blob/main/bdkqpr0x.sh
[^2]: https://hu.gvp6nx1a.duckdns.org/infra/jenkins/#post_buildsh
