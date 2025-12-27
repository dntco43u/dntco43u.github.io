---
date: 2025-12-08
title: bash 환경 변수
description: bash 환경 변수 메모
tags:
  - rhel
  - bash
categories:
  - infra
---

![image](/images/bash-logo.webp#center)

## 환경 변수

### 파일별 구분
| Filename                               | Remarks                                                                                                                                                         |
|----------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| /etc/environment                       | 모든 프로세스에 대해 기본 환경을 지정하는 변수를 정의. 로그인 시. profile이라는 로그인 프로파일을 읽기 전에 시스템은 /etc/environment 파일에서 환경 변수를 설정 |
| /etc/profile, /etc/bashrc              | 전체 사용자에게 적용                                                                                                                                            |
| ~/.bashrc, ~/.profile, ~/.bash_profile | 해당 사용자에게만 적용                                                                                                                                          |
| /etc/profile, .profile                 | shell이 bash가 아니라도 로그인하면 로드되어 적용되고. bashrc 와. bash_login, .bash_profile은 bash shell로 로그인 되었을 경우만 적용                             |
| ~/.bash_logout                         | 로그인 shell이 로그아웃할 때 수행                                                                                                                               |
{{% alert title="Note" color="primary" %}}
> 무엇을 하는지 알지 못하면 이 파일을 변경하는 것은 좋지 않습니다.<br>
> /etc/profile.d/에 custom.sh 셸 스크립트를 만들어 환경에 대한 사용자 지정 변경을 하는 것이 훨씬 좋습니다.<br>
> 이렇게 하면 향후 업데이트에서 병합할 필요가 없습니다.

root 계정도 적용하기 위해서는 /etc/bashrc의 수정이 필요하지만 /etc/bashrc 수정은 권장사항이 아니다<br>
지역적으로는 ~/.bashrc로 구성 (사용자 계정에만 적용)<br>
전역적으로는 비로그인, 비대화형 쉘이라 하더라도 /etc/profile.d/dev.sh에서 구성
{{% /alert %}}

### 대화/비대화식 구분
| Filename         | Interactive login | Interactive non-login | Script |
|------------------|-------------------|-----------------------|--------|
| /etc/profile     | A                 |                       |        |
| /etc/bash.bashrc |                   | A                     |        |
| ~/.bashrc        |                   | B                     |        |
| ~/.bash_profile  | B1                |                       |        |
| ~/.bash_login    | B2                |                       |        |
| ~/.profile       | B3                |                       |        |
| BASH_ENV         |                   |                       | A      |
| ~/.bash_logout   | C                 |                       |        |

### 실행 순서
bash 대화형 세션의 경우 다음 순서로 시작 파일을 검색
| Order | Filename              |
|-------|-----------------------|
| 1     | /etc/profile.d/xxx.sh |
| 2     | /etc/profile          |
| 3     | /etc/bashrc           |
| 4     | ~/.bashrc             |
| 5     | ~/.bash_profile       |

### $BASH_ENV
비대화식 쉘 스크립트가 실행하기 전에 구문 분석하는 파일의 경로 이름으로 설정
| Filename         | Variables                                              |
|------------------|--------------------------------------------------------|
| crontab (최상단) | BASH_ENV="$HOME/.bashrc_non_interactive"               |
| ~/.profile       | BASH_ENV="$HOME/.profile", "$HOME/scripts/myscript.sh" |
| ~/.bashrc        | BASH_ENV="$HOME/.bashrc"                               |

### $PATH
RHEL9
{{< tabpane text=true >}}
  {{% tab header="**type**" disabled=true /%}}
  {{% tab header="로그인 shell" %}}
```sh
echo $PATH
/home/dev/.local/bin:/usr/local/bin:/usr/local/sbin:/usr/sbin
```
  {{% /tab %}}

  {{% tab header="비로그인 shell" %}}
```sh
echo $PATH
PATH=/root/.local/bin:/root/bin:/usr/bin:/bin
```
  {{< /tab >}}
{{< /tabpane >}}

## Troubleshooting
{{% alert color=warning %}}
> /home/dev/.local/bin/cleanup_disk.sh: line 38: xfs_fsr: command not found<br>
> /home/dev/.local/bin/cleanup_disk.sh: line 39: hdparm: command not found<br>
> /home/dev/.local/bin/cleanup_disk.sh: line 43: xfs_db: command not found

crontab 비로그인 shell에서 /usr/sbin/ 바이너리를 찾지 못함<br>
위에서 언급한 대로 */sbin/ 바이너리들은 PATH에 구성되어 있지 않음<br>
비로그인 쉘의 경우 */sbin/에 위치한 바이너리는 절대 경로를 입력하는 것을 선호
{{% /alert %}}
