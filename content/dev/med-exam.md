---
date: 2025-12-09
title: med-exam
description: 건강검진 및 생동성, 임상시험 피실험자에게 공개된 검사 데이터를 가공 후 시각화하고 AI들의 소견 출력
tags:
  - postgresql
  - python
  - grafana
categories:
  - dev
---

![image](/images/grafana-logo.webp#center)

## GitHub repository
[바로 가기](https://github.com/dntco43u/med_exam)

## container 구성

### .env
```sh
vi /opt/python/.env
```
```ini
PGSQL_HOST=host.docker.internal
PGSQL_PORT=5432
PGSQL_DB=dev
PGSQL_USER=dev
PGSQL_PASSWORD=y***************************************************************

PAWAN_API_KEY=p**************************************************
GROQ_API_KEY=g*******************************************************
GEMINI_API_KEY=A**************************************
DEEPL_API_KEY=9**************************************
XWDFELA2_BOT_KEY=7*********************************************

PROXY_HOST=2**.**.**.*
PROXY_PORT=6****
```

### Dockerfile
```sh
vi /opt/python/Dockerfile
```
```dockerfile
FROM python:3-slim
ENV TZ=Asia/Seoul
RUN apt-get update && \
    apt-get install -y python3 && \
    python -m pip install --no-cache-dir --upgrade pip \
    pycryptodome \
    psycopg2-binary \
    requests bs4 lxml \
    selenium \
    deep-translator deepl \
    python-telegram-bot \
    g4f[all] openai groq google-generativeai && \
    apt-get autoremove -y && \
    apt-get clean autoclean && \
    rm -rf /var/lib/apt/lists/*
```
```sh
docker build -t e7hnr8ov/python:3-slim /opt/python
```

### med_exam.sh [^3]
```sh
vi ~/.local/bin/med_exam.sh
```
```sh
#!/bin/bash
# med_exam

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh

container="py-med-exam"
docker ps -q --filter "name=$container" | xargs -r docker stop
sleep 1
docker run \
  -i --rm --name="$container" --network=dev --user=0:0 \
  --add-host=host.docker.internal:host-gateway \
  --env-file=/opt/python/.env \
  -v /home/dev/workspace/med_exam:/opt/python:rw \
  e7hnr8ov/python:3-slim \
  python /opt/python/med_exam.py "$@"
```

## 데모 페이지 [^1]
![image](/images/med-exam-1.webp)
- [바로 가기 1](https://gr.gvp6nx1a.duckdns.org/d/fdyt7rjk3owzkd/med-exam?orgId=1)
- [바로 가기 2](https://agknwpt3.grafana.net/d/fdyt7rjk3owzkd/med-exam?orgId=1&from=2024-07-23T15:00:00.000Z&to=2024-09-09T14:59:59.000Z&timezone=Asia%2FSeoul&var-usr_id=9VFsW2u8mIqFXMlG7f6irg%3D%3D&var-exam_type_cd=002&var-exam_id=RT51KR-PK01&var-rslt_ymd=$__all&var-exam_a_gb_cd=$__all&var-grdng_cd=$__all&var-exam_b_gb_cd=$__all&var-ai_model=$__all)

## Troubleshooting
{{% alert color=info %}}
> python 패키지 g4f 프롬프트 1000자 제한 [^2]

```py
if(re.compile(r"^$|.*{\"code\":[0-9]{3}.*|.*message exceeds.*").match(res)): #빈값, raw값, 메시지 1000자 초과 재시도
```
{{% /alert %}}
{{% alert color=warning %}}
> python 패키지 google-trans-new 번역 불가

패키지 삭제
{{% /alert %}}
{{% alert color=warning %}}
> googletrans==4.0.0-rc1<br>
> 10.38 The conflict is caused by:<br>
> 10.38     googletrans 4.0.0rc1 depends on httpx==0.13.3<br>
> 10.38     groq 0.11.0 depends on httpx<1 and >=0.23.0

python 패키지 googletrans, groq 의존성 충돌
{{% /alert %}}
{{% alert color=warning %}}
> TypeError: not all arguments converted during string formatting

python 패키지 psycopg2 execute 파라미터 마지막 "," 추가
```py
cur.execute("DELETE FROM dev.medi_exam WHERE user_id = %s", (row[j],))
```
{{% /alert %}}
{{% alert color=warning %}}
> SQL Error [0A000]: ERROR: cannot use column reference in DEFAULT expression<br>
> ERROR: cannot use column reference in DEFAULT expression<br>
> ERROR: cannot use column reference in DEFAULT expression

python 패키지 psycopg2 오류. db 컬럼 default값 ''로 변경
{{% /alert %}}
{{% alert color=info %}}
> grafana 절사 오류 (소수점 정밀도 3)

```sql
regexp_replace(trunc(extract(epoch FROM T1.exam_tm - drg_adm_tm) / (60 * 60), 1)::TEXT, '(^[0-9]+)(.0)', '\\1')
```
{{% /alert %}}
{{% alert color=info %}}
> grafana table에서 다른 컬럼값 사용

```json
${__data.fields["prod_link"]}
${__value.raw}
```
{{% /alert %}}

[^1]: https://github.com/dntco43u/med_exam
[^2]: https://discord.com/channels/1205230024970469488/1259285338673643701
[^3]: https://github.com/dntco43u/s6h7k8rv/blob/main/med_exam.sh
