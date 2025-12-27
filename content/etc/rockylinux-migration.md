---
date: 2025-12-07
title: rockylinux 마이그레이션
description: AlmaLinux 8 -> rockylinux 8 마이그레이션
tags:
  - rhel
categories:
  - etc
---

![image](/images/rockylinux-logo.webp#center)

## AlmaLinux 8 -> rockylinux 8 [^1] [^2]
```sh
sudo curl https://raw.githubusercontent.com/rocky-linux/rocky-tools/main/migrate2rocky/migrate2rocky.sh -o migrate2rocky.sh && \
sudo chmod u+x migrate2rocky.sh && \
sudo ./migrate2rocky.sh -r
```
```sh
sudo reboot;
cat /etc/*release*
```

## Troubleshooting
{{% alert color=warning %}}
> Problem 1: package linux-firmware-core-999:20220304-999.13.gitf011ccb4.el8.noarch conflicts with linux-firmware <= 999:20211203-999.10.gitb0e898fb.el8 provided by linux-firmware-20220210-107.git6342082c.el8.noarch

firmware 패키지 충돌. 문제 없음
{{% /alert %}}
{{% alert color=warning %}}
> Problem 2: package tuned-profiles-oci-2.18.0-2.0.1.el8.noarch requires tuned = 2.18.0-2.0.1.el8, but none of the providers can be installed

xoraclecloud 패키지 누락. 문제 없음
{{% /alert %}}
{{% alert color=warning %}}
> Problem 3: package tuned-profiles-oci-2.18.0-2.0.1.el8.noarch requires tuned = 2.18.0-2.0.1.el8, but none of the providers can be installed

oraclecloud 패키지 누락. 문제 없음
{{% /alert %}}

[^1]: https://docs.rockylinux.org/10/guides/migrate2rocky/
[^2]: https://github.com/rocky-linux/rocky-tools/tree/main/migrate2rocky
