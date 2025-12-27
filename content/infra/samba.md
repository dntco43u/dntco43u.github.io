---
date: 2025-12-09
title: samba
description: samba 구성
tags:
  - samba
  - wsdd
categories:
  - infra
---

![image](/images/sh-logo.webp#center)

## host 구성

### 설치
```sh
sudo dnf -y update && \
sudo dnf install -y samba wsdd && \
sudo systemctl enable wsdd && sudo systemctl enable smb && \
sudo systemctl restart wsdd && sudo systemctl restart smb && \
sudo systemctl status wsdd && sudo systemctl status smb
```

### 포트 개방
- smb: 139/tcp, 445/tcp<br>
- wsdd: 3702/udp, 5357/tcp [^1]
```sh
sudo firewall-cmd --permanent --add-port={139,445}/tcp && \
sudo firewall-cmd --permanent --add-port=3702/udp && \
sudo firewall-cmd --permanent --add-port=5357/tcp && \
sudo firewall-cmd --reload && \
sudo firewall-cmd --list-all
```

### selinux
```sh
sudo setsebool -P samba_export_all_ro on && \
sudo setsebool -P samba_export_all_rw on && \
sudo chcon -R -t samba_share_t /mnt/d2 && \
ls -lZ /mnt/d2 && \
sudo semanage boolean -l | grep samba
```

### smb.conf
```sh
vi /etc/samba/smb.conf
```
```ini
...
[global]
  workgroup = WORKGROUP
  server min protocol = SMB3
  client min protocol = SMB3
  security = USER
  wins support = yes

[d2]
  path = /mnt/d2
  guest ok = no
  writable = yes
  valid users = dev
  create mask = 0660
  directory mask = 0770
```

### wsdd
```sh
sudo vi /usr/lib/systemd/system/wsdd.service
```
```ini
# Command-line options for wsdd
OPTIONS="--interface eth0 --ipv4only --shortlog"
```

### 암호, 권한 구성
```sh
sudo chown -R dev:dev /mnt/d2 && \
sudo chmod -R 0660 /mnt/d2 && \
sudo chmod -R ug+X /mnt/d2 && \
sudo smbpasswd -a dev
*****************
sudo systemctl restart smb
```

## License
상업적 이용 제한 없음
- samba: GNU GPL v3 [^4]
- wsdd: MIT [^5]

## Troubleshooting
{{% alert color=warning %}}
> 네트워크에 samba가 노출되지 않음

wsdd 서비스 확인. nic 이름이 변경된 경우 wsdd 구성 변경 필요
```sh
sudo systemctl status wsdd && sudo systemctl status smb
```
{{% /alert %}}

{{% alert color=warning %}}
> 원본과 달리 samba에서 8.3형식으로 파일명이 변경됨 [^2] [^3]

파일명이 samba 표준과 어긋남 (후행 공백 등)
```sh
vi /etc/samba/smb.conf
```
```ini
...
[global]
  mangled names = no
...
```
samba 구성 변경으로 접근하기보다 원본 폴더명을 변경하는 식으로 해결할 것
{{% /alert %}}

## References
- https://unix.stackexchange.com/questions/600026/samba-share-with-force-user-is-still-not-writable-unless-files-have-777-permis/750152#750152
- https://forum.level1techs.com/t/synology-ds1618-performance-tweaking-enabling-smb-multichannel/138719/1
- https://superuser.com/questions/1587290/why-is-smb-client-not-capable-of-using-rss-although-its-enabled
- https://docs.microsoft.com/en-us/archive/blogs/josebda/the-basics-of-smb-multichannel-a-feature-of-windows-server-2012-and-smb-3-0
- https://lokna.no/?p=1617
- https://en.wikipedia.org/wiki/Samba_(software)

[^1]: https://github.com/christgau/wsdd
[^2]: https://superuser.com/questions/458995/files-folders-get-weird-names-and-become-inaccessible-on-samba-share
[^3]: https://forums.truenas.com/t/smb-mangles-long-file-names-to-dos8-3-why-have-i-fixed-it/10784/10
[^4]: https://www.samba.org/samba/ms_license.html
[^5]: https://github.com/christgau/wsdd/blob/master/LICENSE
