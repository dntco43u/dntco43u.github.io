---
date: 2025-12-08
title: oracle/instantclient
description: oracle 19 커넥터 구성
tags:
  - docker
  - oracle
categories:
  - infra
---

![image](/images/oracle-logo.webp#center)

## container 구성

### Dockerfile (x86)
```sh
sudo vi /opt/instantclient/Dockerfile
```
```dockerfile
#FIXME: ubuntu 최신 버전 libaio1 없으므로 22.04로 build
FROM ubuntu:22.04
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Seoul
ENV ORACLE_HOME=/opt/oracle/instantclient
ENV PATH=$PATH:$ORACLE_HOME
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME
ENV SQLPATH=${INSTANT_CLIENT_PATH}:${SQLPATH}
ENV TNS_ADMIN=/opt/oracle/instantclient/network/admin
RUN apt-get update && \
    apt-get install -y tzdata \
    libaio1 \
    python3 \
    python3-pip \
    curl \
    unzip && \
    ln -sf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    echo /opt/oracle/instantclient > /etc/ld.so.conf.d/oic.conf && \
    ldconfig && \
    curl https://download.oracle.com/otn_software/linux/instantclient/1919000/instantclient-basiclite-linux.x64-19.19.0.0.0dbru.el9.zip -so /tmp/instantclient.zip && \
    unzip -o /tmp/instantclient.zip -d /opt/oracle && \
    rm /tmp/instantclient.zip && \
    curl https://download.oracle.com/otn_software/linux/instantclient/1919000/instantclient-sqlplus-linux.x64-19.19.0.0.0dbru.el9.zip -so /tmp/instantclient.zip && \
    unzip -o /tmp/instantclient.zip -d /opt/oracle && \
    rm /tmp/instantclient.zip && \
    curl https://download.oracle.com/otn_software/linux/instantclient/1919000/instantclient-tools-linux.x64-19.19.0.0.0dbru.el9.zip -so /tmp/instantclient.zip && \
    unzip -o /tmp/instantclient.zip -d /opt/oracle && \
    rm /tmp/instantclient.zip && \
    cd /opt/oracle && \
    mv instantclient_19_19 instantclient && \
    python3 -m pip install --no-cache-dir pip && \
    python3 -m pip install --no-cache-dir oracledb && \
    apt-get purge -y python3-pip \
    curl \
    unzip && \
    apt-get autoremove -y && \
    apt-get clean autoclean && \
    rm -rf /var/lib/apt/lists/*
```
```sh
docker build -t e7hnr8ov/oracle-instantclient:alpine /opt/instantclient
```

### Dockerfile (arm)
```sh
sudo vi /opt/instantclient/Dockerfile
```
```dockerfile
FROM python:alpine
ENV TZ Asia/Seoul
ENV LD_LIBRARY_PATH /opt/oracle/instantclient
RUN apk update && \
    apk add --no-cache g++ \
    libaio \
    libc6-compat \
    curl && \
    curl https://download.oracle.com/otn_software/linux/instantclient/191000/instantclient-basiclite-linux.arm64-19.10.0.0.0dbru.zip -so /tmp/instantclient.zip && \
    unzip -o /tmp/instantclient.zip -d /opt/oracle && \
    cd /opt/oracle && \
    mv instantclient_19_10 instantclient && \
    ln -s /opt/oracle/instantclient/libclntsh.so.19.1 /usr/lib/libclntsh.so && \
    ln -s /opt/oracle/instantclient/libocci.so.19.1 /usr/lib/libocci.so && \
    ln -s /opt/oracle/instantclient/libociicus.so /usr/lib/libociicus.so && \
    ln -s /opt/oracle/instantclient/libnnz19.so /usr/lib/libnnz19.so && \
    ln -s /lib/libc.so.6 /usr/lib/libresolv.so.2 && \
    rm /tmp/instantclient.zip && \
    python -m pip install --no-cache-dir cx_Oracle && \
    apk del curl && \
    rm /var/cache/apk/*
```
```sh
docker build -t e7hnr8ov/oracle-instantclient:alpine /opt/instantclient
```

## host 구성

### crond
일정 기간 db 유휴 시 데이터 삭제를 공지하고 있으므로 매일 테스트 스키마를 갱신하도록 구성 [^1] [^2]
{{< tabpane text=true >}}
  {{% tab header="**sh**" disabled=true /%}}
  {{% tab header="cqa7wtjg_helper.sh" %}}
```sh
#!/bin/bash
# oci oralcedb 유휴 방지

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

user=HR
pass=z*****************************
url="'(description= (retry_count=20)(retry_delay=3)(address=(protocol=tcps)(port=1521)(host=adb.ap-seoul-1.oraclecloud.com))(connect_data=(service_name=g**************_********_tpurgent.adb.oraclecloud.com))(security=(ssl_server_dn_match=yes)))'"
docker run \
  -i --rm --name=instantclient-cqa7wtjg --network=dev --user=0:0 \
  --env-file /opt/instantclient/.env \
  -e TZ=Asia/Seoul \
  -v /opt/instantclient/config/wallet_cqa7wtjg:/opt/oracle/instantclient/network/admin:ro \
  -v /opt/instantclient/data:/data:rw \
  e7hnr8ov/instantclient:ubuntu \
  /bin/bash -c "exit | sqlplus $user/$pass@$url @/data/hr_schema/drop.sql" > "$log_file"
tail -n4 "$log_file" | head -n1 > "$msg_file"

docker run \
  -i --rm --name=instantclient-cqa7wtjg --network=dev --user=0:0 \
  --env-file /opt/instantclient/.env \
  -e TZ=Asia/Seoul \
  -v /opt/instantclient/config/wallet_cqa7wtjg:/opt/oracle/instantclient/network/admin:ro \
  -v /opt/instantclient/data:/data:rw \
  e7hnr8ov/instantclient:ubuntu \
  /bin/bash -c "exit | sqlplus $user/$pass@$url @/data/hr_schema/make.sql" >> "$log_file"
tail -n4 "$log_file" | head -n1 >> "$msg_file"

send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
rm "$msg_file"
```
  {{% /tab %}}

  {{% tab header="dwc8khum_helper.sh" %}}
```sh
#!/bin/bash
# oci oralcedb 유휴 방지

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

user=HR
pass=B*****************************
url="'(description= (retry_count=20)(retry_delay=3)(address=(protocol=tcps)(port=1521)(host=adb.ap-seoul-1.oraclecloud.com))(connect_data=(service_name=g**************_********_high.adb.oraclecloud.com))(security=(ssl_server_dn_match=yes)))'"
docker run \
  -i --rm --name=instantclient-dwc8khum --network=dev --user=0:0 \
  --env-file /opt/instantclient/.env \
  -e TZ=Asia/Seoul \
  -v /opt/instantclient/config/wallet_dwc8khum:/opt/oracle/instantclient/network/admin:ro \
  -v /opt/instantclient/data:/data:rw \
  e7hnr8ov/instantclient:ubuntu \
  /bin/bash -c "exit | sqlplus $user/$pass@$url @/data/hr_schema/drop.sql" > "$log_file"
tail -n4 "$log_file" | head -n1 > "$msg_file"

docker run \
  -i --rm --name=instantclient-dwc8khum --network=dev --user=0:0 \
  --env-file /opt/instantclient/.env \
  -e TZ=Asia/Seoul \
  -v /opt/instantclient/config/wallet_dwc8khum:/opt/oracle/instantclient/network/admin:ro \
  -v /opt/instantclient/data:/data:rw \
  e7hnr8ov/instantclient:ubuntu \
  /bin/bash -c "exit | sqlplus $user/$pass@$url @/data/hr_schema/make.sql" >> "$log_file"
tail -n4 "$log_file" | head -n1 >> "$msg_file"

send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
rm "$msg_file"
```
  {{< /tab >}}
{{< /tabpane >}}

## Troubleshooting
{{% alert color=info %}}
thick으로 연결 시 wallet 여부 관계 없이 `/opt/oracle/instantclient/network/admin` mount 필수
{{% /alert %}}

{{% alert color=info %}}
> sqlplus 실행 후 종료

파이프 추가. 아래는 bash 예
```sh
/bin/bash -c "exit | sqlplus HR/z******************************'(description= (retry_count=20)(retry_delay=3)(address=(protocol=tcps)(port=1521)(host=adb.ap-seoul-1.oraclecloud.com))(connect_data=(service_name=g**************_cqa7wtjg_tpurgent.adb.oraclecloud.com))(security=(ssl_server_dn_match=yes)))' @/data/hr_schema/make.sql"
```
{{% /alert %}}

{{% alert color=warning %}}
> Package 'libaio1' has no installation candidate

> libaio1 패키지는 버전 24.04부터 libaio1t64로 대체되었습니다.<br>
> Oracle은 종속성을 libaio1t64로 업데이트하여 SQLPLUS를 다시 컴파일해야 합니다.<br>
> 현재 작동하려면 @simon-dba 에서 제안한대로 libaio1t64 -> libaio1 에서 심볼릭 링크를 만들어야합니다.

ubuntu v24는 symlink 생성 `libaio1` -> `libaio1t64` [^3]
```sh
apt-get update -y && apt-get intsall -y libaio1 && \
apt-get update -y && apt-cache search libaio1 && \
ln -s /usr/lib/x86_64-linux-gnu/libaio.so.1t64 /opt/oracle/instantclient_19_19/libaio.so.1
```
{{% /alert %}}

## License
상업적 이용 제한 없음 [^4]
> Q. Instant Client 비용은 얼마인가요?<br>
> A. Instant Client는 OTN에서 무료로 제공되어 개발 또는 운영 환경에서 누구나 사용할 수 있습니다.<br>
> 하지만 고객은 이미 표준 지원 계약이 있는 경우에만 Oracle 지원에 연락할 수 있습니다. [^5]

## References
- https://www.oracle.com/database/technologies/instant-client/downloads.html
- https://medium.com/db-one/oracle-jdbc-drivers-a84e2b5f7eb9

[^1]: https://github.com/dntco43u/s6h7k8rv/blob/main/cqa7wtjg_helper.sh
[^2]: https://github.com/dntco43u/s6h7k8rv/blob/main/dwc8khum_helper.sh
[^3]: https://forums.oracle.com/ords/apexds/post/instant-client-on-ubuntu-24-04-noble-numbat-7244
[^4]: https://www.oracle.com/downloads/licenses/instant-client-lic.html
[^5]: https://www.oracle.com/kr/database/technologies/faq-instant-client.html
