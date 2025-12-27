---
date: 2025-12-07
title: RHEL9 기타 구성
description: RHLE9 기타 구성 메모
tags:
  - rhel
categories:
  - infra
---

![image](/images/rhel-logo.webp#center)

## 기타 구성

### ~~swap~~
생성
```sh
sudo fallocate -l 4G /swapfile && \
sudo chmod 600 /swapfile && \
sudo mkswap /swapfile && \
sudo swapon /swapfile && \
sudo tee -a /etc/fstab <<EOF
UUID=23ff5ad2-345b-45a4-992d-71d4a1a99f9b /         ext4 rw, discard, errors=remount-ro, x-systemd.growfs, noatime, nodiratime 0 1
UUID=5D92-027D                            /boot/efi vfat defaults                                                         0 0
/swapfile                                 swap      swap defaults                                                         0 0
EOF
```
```sh
sudo mount -a
```

변경
```sh
sudo dd if=/dev/zero of=/swapfile bs=1MiB count=$((4*1024)) && \
sudo chmod 600 /swapfile && \
sudo mkswap /swapfile && \
sudo swapon -v /swapfile && \
sudo swapoff -v /previous_swapfile && \
sudo rm /previous_swapfile
```

### ~~zswap~~
속도 저하 체감으로 삭제
```sh
sudo grubby --update-kernel=/boot/vmlinuz-$(uname -r) --args="zswap.enabled=0" && sudo reboot
```
```sh
sudo grep -r . /sys/kernel/debug/zswap
```

### zram [^1]
swap, zswap에서 옮겨온 구성
```sh
echo "zram" | sudo tee /etc/modules-load.d/zram.conf && \
echo "options zram num_devices=1" | sudo tee /etc/modprobe.d/zram.conf && \
echo KERNEL==\"zram0\", ATTR{disksize}=\"8G\",TAG+=\"systemd\" | sudo tee /etc/udev/rules.d/99-zram.rules
```
```sh
sudo vi /etc/systemd/system/zram.service
```
```ini
[Unit]
Description=Swap with zram
After=multi-user.target

[Service]
Type=oneshot
RemainAfterExit=true
ExecStartPre=/sbin/mkswap /dev/zram0
ExecStart=/sbin/swapon /dev/zram0
ExecStop=/sbin/swapoff /dev/zram0

[Install]
WantedBy=multi-user.target
```
```sh
sudo systemctl enable zram.service && \
sudo systemctl restart zram.service && \
sudo systemctl restart zram-swap.service && \
sudo reboot;
zramctl
```

### 포트 개방
```sh
sudo firewall-cmd --set-default-zone=public && \
sudo firewall-cmd --get-active-zones && \
sudo firewall-cmd --permanent --zone=public --add-masquerade && \
sudo firewall-cmd --permanent --remove-service=cockpit && \
sudo firewall-cmd --permanent --remove-service=dhcpv6-client
```
```sh
sudo firewall-cmd --permanent --remove-service=ssh && \
sudo firewall-cmd --permanent --add-forward-port=port=6****:proto=tcp:toport=22 && \
sudo firewall-cmd --reload && \
sudo firewall-cmd --list-all
```

### 저널 로그 정책
```sh
sudo journalctl --vacuum-time=7d && \
sudo journalctl --vacuum-size=100M && \
sudo systemctl restart systemd-journald;
```

### 이전 커널 삭제
```sh
sudo dnf -y update && sudo dnf -y remove --oldinstallonly --setopt installonly_limit=2 kernel
```

### 불필요 패키지 삭제
```sh
sudo dnf remove -y insights-client cockpit* && \
subscription-manager remove --all && \
subscription-manager unregister && \
subscription-manager clean
```

### TCP BBR
```sh
sudo tee -a /etc/sysctl.conf <<EOF
vm.swappiness=1
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
net.core.busy_read=50
net.core.busy_poll=50
EOF
```
```sh
sudo sysctl -p
```

### selinux 비활성화
```sh
sudo vi /etc/sysconfig/selinux
```
```ini
SELINUX=permissive
```
```sh
sudo setenforce 0 && \
sudo reboot;
getenforce
```

### nic 이름 변경
관리 편의와 수집 데이터 통일을 위해 변경 (enp4s0 -> eth0) [^2]
```sh
cat /sys/class/net/enp4s0/type
```
```sh
cat << EOF | sudo tee /etc/udev/rules.d/70-persistent-net.rules
SUBSYSTEM=="net",ACTION=="add",ATTR{address}=="00:d8:61:1c:eb:65",ATTR{type}=="1",NAME="eth0"
EOF
```
```sh
sudo nmcli connection modify enp4s0 connection.interface-name "" && \
sudo nmcli connection modify enp2s0 match.interface-name "eth0" && \
sudo reboot
```
```sh
ip link show && \
nmcli -f device,name connection show && \
sudo nmcli connection up "enp2s0"
```

### wakeonlan
```sh
sudo ethtool -s eth0 wol g && \
sudo ethtool eth0 | grep Wake-on
```

### 절전 구성
baremetal 서버만 (optional)
```sh
grep -i pstate /boot/config-$(uname -r) && \
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_driver
```
hwp 지원 intel cpu는 cmos에서 제어하거나 자체 sysfs file로 구성할 것
```sh
cpupower frequency-info && \
available cpufreq governors: conservative ondemand userspace powersave performance schedutil && \
sudo cpupower frequency-set --governor powersave && \
sudo systemctl enable cpupower
```
```sh
ls /sys/devices/system/cpu/intel_pstate
```
```sh
sudo dnf install -y powertop tuned-utils && \
sudo powertop && \
sudo systemctl enable powertop && \
sudo powertop2tuned dev -ef && \
sudo tuned-adm profile dev
```

### 커널 psi
```sh
sudo grubby --update-kernel=ALL --args="psi=1" && \
sudo reboot;
ls /proc/pressure
```
{{% alert title="Note" color=primary %}}
> PSI는 리소스가 개발됨에 따라 리소스 부족이 증가하는 것을 확인할 수 있는 표준 방법을 제공합니다.<br>
> 메모리, CPU 및 I/O(출력/출력)의 세 가지 주요 리소스에 대한 압력 지표가 있습니다.<br>
> PSI는 RHEL 8 이상 버전에서 사용할 수 있으며 기본적으로 비활성화되어 있습니다.<br>
> PSI가 활성화되면 리소스 최적화 서비스가 결과를 보강하고 더 자세한 내용과 더 나은 제안을 제공할 수 있습니다.<br>
> PSI를 활성화하면 최대 값을 식별하는 것이 좋습니다.<br>
> PSI를 사용하도록 설정하면 약간의 (<1%) 성능이 저하됩니다.

Node Exporter Full에서 psi 정보 수집 [^8]
{{% /alert %}}


### epel 저장소
```sh
sudo dnf -y update && sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

### 자주 쓰는 유틸리티
```sh
sudo dnf -y update && sudo dnf install -y hdparm p7zip p7zip-plugins bind-utils rsync
```

### shellcheck
```sh
sudo dnf -y update && sudo dnf install -y shellcheck
```
사용자 shell 전체 검사 [^4]
```sh
find ~/.local/bin/ -type f -iname *.sh -exec shellcheck -x {} \;
```

### aws
계정 변경 (ec2-user -> dev)
```sh
sudo tee -a ~/clone_user.sh <<EOF
#!/bin/bash
src=$1
dest=$2
src_groups=$(id -Gn "$src" | sed "s/ /,/g" | sed -r 's/\<'"$src"'\>\b,?//g')
src_shell=$(awk -F : -v name="$src" '(name == $1) { print $7 }' /etc/passwd)
sudo useradd --groups "$src_groups" --shell "$src_shell" --create-home "$dest"
sudo passwd "$dest"
EOF
```
```sh
sudo chmod 700 ~/clone_user.sh && \
sudo ~/clone_user.sh $(id -nu) dev && \
sudo cp -R /home/$(id -nu)/.ssh /home/dev && \
sudo chown dev:dev -R /home/dev && \
sudo rm /home/ec2-user/clone_user.sh
```
```sh
echo "%wheel ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/wheel && \
sudo usermod -aG wheel dev
```
user process 실행 중이면 kill하고 진행
```sh
sudo usermod -u 1003 ec2-user && sudo groupmod -g 1003 ec2-user && \
sudo cat /etc/passwd | grep ec2-user && \
sudo usermod -u 1000 dev && sudo groupmod -o -g 1000 dev && \
sudo cat /etc/passwd | grep dev && \
sudo chown -R dev:dev /home/dev && \
sudo usermod -u 1001 ec2-user && sudo groupmod -g 1001 ec2-user && \
sudo cat /etc/passwd | grep ec2-user && \
sudo chown 1001:1001 -R /home/ec2-user
```
hostname 구성
```sh
echo "rqzviph1" | sudo tee /etc/hostname
logout
```

### bash profile
전역 변수 구성
```sh
sudo tee /etc/profile.d/dev.sh <<EOF
BASH_ENV=/etc/profile.d/dev.sh
PS1="\[\033[36m\]\u\[\033[0m\]@\[\033[32m\]\h\[\033[m\]:\[\033[33;1m\]\w\[\033[m\]\$ "
TEL_BOT_KEY="6*********************************************"
TEL_CHAT_ID="1*********"
EOF
```

### motd
```sh
sudo vi /etc/motd.d/dev
```
```sh
[0m
[0m [0m [0m [0m [0m [0m [0m [0m [32m.[32m,[32m:[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m,[32m.[0m [32m.[32m,[32m;[32mc[32mc[32mc[32mc[32mc[32m:[32m'[0m [0m [0m [0m [0m [0m [0m [32m[0m
[0m [0m [0m [0m [0m [0m [32m.[32m;[32mc[32mc[32mc[32mc[32m:[32m;[32m;[32m;[32m:[32m:[32m:[32m;[32m:[32m:[32m,[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m:[0m [0m [0m [0m [0m [0m [32m[0m
[0m [0m [0m [0m [0m [32m.[32mc[32mc[32mc[32mc[32m;[32m;[32m:[0mo[0mk[0mk[0mk[0mk[0mx[0mo[0mc[0m;[32m'[32m,[32m:[32m;[32m;[32m;[32m;[32m;[32m:[32m;[32m;[32m'[32m.[0m
[0m [0m [0m [0m [32m.[32mc[32mc[32mc[32mc[32m:[0ml[0mO[0mW[0mM[0mM[0m0[0m.[0m.[0m.[0m'[0ml[0mN[0mK[0mo[0m,[32m;[32mc[0mO[0mX[0mM[0mM[0mM[0mK[0mx[0mx[0m0[0mk[0m,[0m [0m
[0m [32m.[32m.[32mc[32mc[32mc[0mc[0mx[0mK[0mM[0mM[0mM[0mM[0mM[0mc[0m [0m [0m [0m.[0m [0mc[0mM[0mM[0mW[0md[0mk[0mM[0mM[0mM[0mM[0m:[0m [0m [0m.[0m.[0mk[0mM[0m0[0m.[0m
[32mc[32mc[32m;[32mc[32mc[32mc[32mc[32m;[0mk[0mM[0mM[0mM[0mM[0mM[0mW[0mx[0mc[0m,[0m,[0ml[0mN[0mM[0mM[0mM[0mM[0mO[0mM[0mM[0mM[0mM[0mk[0m.[0m.[0m [0m [0mo[0mM[0mM[0md[0m
[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m;[0m:[0md[0mK[0mW[0mM[0mM[0mM[0mM[0mM[0mM[0mM[0mM[0mM[0mM[0mM[0mM[0mO[0mM[0mM[0mM[0mM[0mM[0mM[0mN[0mX[0mN[0mM[0mM[0mK[0m.[0m
[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m:[32m;[32m;[0m:[0mc[0md[0mk[0m0[0m0[0m0[0m0[0mk[0mO[0mx[0m:[0ml[0mN[0mM[0mM[0mM[0mM[0mM[0mM[0mM[0mN[0mk[0m,[0m [0m [0m
[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m;[32m;[32m;[32mc[32mc[32mc[32mc[32mc[32mc[32m:[32m:[32m:[32m:[32m:[32m:[32mc[32mc[32mc[32mc[32mc[32m;[32m;[32m:[32m:[32mc[32mc[32mc[32m;[0m [0m [0m [0m [0m [32m[0m
[32mc[32mc[32mc[32mc[32mc[32mc[32m'[31m;[31m:[31m:[32m,[32m;[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m:[32mc[32mc[32mc[32mc[32m:[32m.[0m [0m [0m [32m[0m
[32m:[32m:[32mc[32mc[32mc[32mc[32m,[31m;[31m:[31m,[31m,[31m;[31m,[32m,[32m;[32m;[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m.[0m [0m [32m[0m
[32m,[32m,[32m:[32m,[32m'[32m:[32m;[32m'[32m,[31m;[31m:[31m,[31m,[31m,[31m;[31m;[31m;[31m,[32m;[32m;[32m;[32m;[32m:[32m:[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m:[32m:[32m;[32m;[32m,[32m.[32m[0m
[32mc[32m;[32m;[32mc[32m'[32m,[32mc[32m,[32mc[32m;[32m,[31m,[31m,[31m:[31m:[31m;[31m,[31m,[31m,[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[0m [0m [32m[0m
[32mc[32mc[32mc[32mc[32m;[32m:[32mc[32m,[32mc[32mc[32mc[32mc[32mc[32m;[32m;[32m;[32m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m;[31m:[31m,[32m:[32m;[32m.[32m[0m
[32m;[32mc[32mc[32mc[32mc[32mc[32mc[32m:[32m,[32m;[32m;[32m;[32m:[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m:[32m;[32m;[32m;[32m;[32m;[32m;[32m;[32m;[32m;[32m;[32m,[32m;[32m:[32m,[32m.[32m[0m
[34m.[32m'[32m;[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m;[32m;[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m:[32m;[32m;[32m;[32m;[32m:[32m;[34m,[0m [0m [0m [0m [0m [32m[0m
[34m'[34m'[34m'[34m.[32m,[32mc[32mc[32mc[32mc[32mc[32mc[32mc[32m,[32m,[32m;[32m;[32m;[32m;[32m;[32m;[32m;[32m;[32m;[32m,[32m'[34m.[34m,[34m,[34m'[34m'[34m'[34m'[34m'[34m'[34m'[34m [0m [0m [0m [0m [32m[0m
[34m'[34m'[34m'[34m.[34m'[34m'[34m'[34m,[34m,[34m,[34m,[34m'[34m.[34m.[34m.[34m'[34m'[34m'[34m'[34m'[34m'[34m.[34m.[34m'[34m'[34m'[34m.[34m'[34m'[34m'[34m'[34m'[34m'[34m'[34m.[0m [0m [0m [0m [0m
[0m
```

### logrotate
사용자 log
```sh
sudo vi /etc/logrotate.d/dev
```
```
/home/dev/.local/log/*.log {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  nocompress
  create 0664 dev dev
  sharedscripts
  postrotate
    docker restart promtail >/dev/null 2>&1 || true
  endscript
}
```

이하 기본 logrotate 재구성
```sh
sudo vi /etc/logrotate.d/btmp
```
```
/var/log/btmp {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  create 0660 root utmp
}
```

```sh
sudo vi /etc/logrotate.d/chrony
```
```
/var/log/chrony/*.log {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  sharedscripts
  postrotate
    /usr/bin/chronyc cyclelogs > /dev/null 2>&1 || true
  endscript
  nocreate
}
```

```sh
sudo vi /etc/logrotate.d/dnf
```
```
/var/log/hawkey.log {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  create
}
```

```sh
sudo vi /etc/logrotate.d/firewalld
```
```
/var/log/firewalld {
  weekly
  missingok
  rotate 4
  copytruncate
  minsize 1M
}
```

```sh
sudo vi /etc/logrotate.d/iscsiuiolog
```
```
/var/log/iscsiuio.log {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  sharedscripts
  postrotate
    pkill -USR1 iscsiuio 2> /dev/null || true
  endscript
}
```

```sh
sudo vi /etc/logrotate.d/kvm_stat
```
```
/var/log/kvm_stat.csv {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  postrotate
    /usr/bin/systemctl try-restart kvm_stat.service
  endscript
}
```

```sh
sudo vi /etc/logrotate.d/rsyslog
```
```
/var/log/cron
/var/log/maillog
/var/log/messages
/var/log/secure
/var/log/spooler {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  create 0664 root root
  sharedscripts
  postrotate
    /usr/bin/systemctl -s HUP kill rsyslog.service >/dev/null 2>&1 || true
    docker restart promtail >/dev/null 2>&1 || true
  endscript
}
```

```sh
sudo vi /etc/logrotate.d/sssd
```
```
/var/log/sssd/*.log {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  sharedscripts
  postrotate
    /bin/kill -HUP `cat /var/run/sssd.pid 2>/dev/null` 2> /dev/null || true
    /bin/pkill -HUP sssd_kcm 2> /dev/null || true
  endscript
}
```

```sh
sudo vi /etc/logrotate.d/wtmp
```
```
/var/log/wtmp {
  daily
  rotate 7
  missingok
  notifempty
  dateext
  dateyesterday
  dateformat -%Y%m%d
  create 0664 root utmp
}
```
<br>
전체 구성 테스트

```sh
sudo find /etc/logrotate.d/ -type f -exec w {} \;
```
selinux
```sh
sudo vi /etc/audit/auditd.conf
```
```
...
local_events = yes
write_logs = yes
log_file = /var/log/audit/audit.log
log_group = root
log_format = ENRICHED
flush = INCREMENTAL_ASYNC
freq = 50
max_log_file = 8
num_logs = 5
...
```

### crond [^5] [^6] [^7]
```sh
vi ~/.local/bin/cleanup_disk.sh
```
```sh
#!/bin/bash
# 디스크 정리

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

disk="$1"
days="$2"
if [ -z "$disk" ] || [ -z "$days" ]; then
  err "MANDATORY PARAMETER MISSING"
  exit 1
fi
{ #$days 이상 지난 log 삭제
  varlog_cnt=$(find /var/log/ -type f -mtime +"$days" | wc -l)
  usrlog_cnt=$(find /home/dev/.local/log/ -type f -mtime +"$days" | wc -l)
  docker_cnt=$(find /opt/ -type f \
    -regex ".*\.\(log\|log\.[0-9]*.*\|log\.bak.*\)" \
    -mtime +"$days" | wc -l)
  find /var/log/ -type f -mtime +"$days" -exec rm {} \;
  find /home/dev/.local/log/ -type f -mtime +"$days" -exec rm {} \;
  find /opt/ -type f \
    -regex ".*\.\(log\|log\.[0-9]*.*\|log\.bak.*\)" \
    -mtime +"$days" -exec rm {} \;

  #docker code-server 폴더 로그 삭제
  if [ -d /opt/code-server ]; then
    base_date=$(date +%Y%m%d -d "$days day ago")
    delete_over_date "$base_date" "/opt/code-server/config/data/logs" \
      "cut -c 1-8"
  fi

  #docker 미사용 이미지 삭제
  read -ra args < <(docker images -q -f dangling=true)
  docker rmi "${args[@]}"

  #prrometheus wal 삭제 (임시)
  #rm -rf /opt/prometheus/data/wal/*
  #rm -rf /opt/prometheus/data/chuncks_head/*
  #nginx log 삭제 (임시)
  #truncate -s 0 /opt/nginx/log/*.log

  #xfs 조각 모음
  /usr/sbin/xfs_fsr "$disk";
  /usr/sbin/hdparm -Tt "$disk"
} > "$log_file"

{ echo "$disk"
  #xfs 조각화 수준 검사
  /usr/sbin/xfs_db -c frag -r "$disk" | grep -o "fragmentation factor.*" \
    | sed -E 's/(fragmentation) (factor) (.*)/\1: \3/g'
  grep "reads" "$log_file" -a \
    | sed -E 's/^(.*)(cached|disk)( reads: )(.* = )(.*)/\2\3\5/g' \
    | sed 's/  */ /g'
  echo "mtime +$days logs: $(("$docker_cnt"+"$varlog_cnt"+"$usrlog_cnt"))"
} > "$msg_file"
send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
rm "$msg_file"
```
```sh
vi ~/.local/bin/dpasswd.sh
```
```sh
#!/bin/bash
# .dpasswd 파일 생성

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

db_yn="$1" #db 연계 생성 여부
if [ -z "$db_yn" ]; then
  err "MANDATORY PARAMETER MISSING"
  exit 1
fi

#.dpasswd 하루 1회 생성
file_mtime=$(stat --printf="%y" /home/dev/.local/etc/.dpasswd)
file_date=$(cut -c 1-10 <<< "$file_mtime" | sed 's/-//g')
if [[ $(date +%Y%m%d) == "$file_date" ]]; then
  echo "GENERATED ONLY ONCE PER DAY"
  exit 0
fi
mv /home/dev/.local/etc/.dpasswd "/home/dev/.local/etc/.dpasswd-$file_date"

if [ "$db_yn" == Y ]; then
  docker run \
    -i --rm --name=instantclient-icnex2pa --network=dev --user=0:0 \
    --env-file /opt/instantclient/.env \
    -e TZ=Asia/Seoul \
    -v /opt/instantclient/data:/data:rw \
    e7hnr8ov/instantclient:ubuntu \
    python3 /data/icnex2pa.py > .dpasswd.tmp
  sed 's/key=//g' .dpasswd.tmp | sed -z 's/\n//g' > /home/dev/.local/etc/.dpasswd
  rm .dpasswd.tmp
else
  rand_string 64 > /home/dev/.local/etc/.dpasswd
fi
echo "Secret key updated" > "$log_file"
show_secret "$(< /home/dev/.local/etc/.dpasswd)" 8 >> "$log_file"

#backup to sj9n7air-ftp
if [ -f "/home/dev/.local/etc/.dpasswd-$file_date" ]; then
  rclone copy \
    --config /home/dev/.config/rclone/rclone.conf \
    --log-file /home/dev/.local/log/rclone_dpasswd.log --log-level DEBUG \
    "/home/dev/.local/etc/.dpasswd-$file_date" \
    sj9n7air-ftp:/mnt/d2/backups/fhy8vp3u/etc
fi
rm /home/dev/.local/etc/.dpasswd-*

echo "key=$(< /home/dev/.local/etc/.dpasswd)" > "$msg_file"
send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
rm "$msg_file"
```
```sh
vi ~/.local/bin/notify_df.sh
```
```sh
#!/bin/bash
# disk free 알림

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

if [ -z "$1" ]; then
  err "MANDATORY PARAMETER MISSING"
  exit 1
fi
{ for i in $1; do
    show_disk_stat "$i";
  done;
} > "$log_file"
cp "$log_file" "$msg_file"
send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
rm "$msg_file"
```

## License
상업적 이용 제한 없음
- rockylinux: Begin 3-Clause BSD License [^9]

## Troubleshooting
{{% alert color=warning %}}
> /usr/sbin/grub2-probe: error: failed to get canonical path of '/dev/mapper/rl_sj9n7air-root'

업데이트 안 되는 경우 boot menu에서 args를 새 이름으로 변경하고 진입 후 grub2-mkconfig 실행
```sh
sudo grubby --info DEFAULT && \
sudo grubby args="ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M noresume rd.lvm.lv=rl/root psi=1" root="/dev/mapper/rl-root" --update-kernel ALL
```
{{% /alert %}}

{{% alert color=warning %}}
> ```sh
> sudo journalctl -b -u auditd
> ```
> /var/log/audit/audit.log is not a regular file<br>
> The audit daemon is exiting.
```sh
sudo restorecon -Rv /var/log/audit/ && \
sudo rm -rf /var/log/audit/audit.log && \
sudo touch /var/log/audit/audit.log && \
sudo chown root:root /var/log/audit && \
sudo chmod 0700 /var/log/audit && \
sudo chmod 0600 /var/log/audit/audit.log
sudo service auditd restart && sudo systemctl status auditd
```
audit service 중지되었을 때 systemctl로는 다시 시작 불가. service 사용할 것
{{% /alert %}}

## References
- https://askubuntu.com/questions/290937/what-is-the-best-directory-for-script-files
- https://superuser.com/questions/981981/how-can-i-excute-my-bash-shell-script-from-any-directory

[^1]: [최소: 8GB, 최대: 총 메모리/2 (zram-size=min(ram/2, 8192))](https://www.reddit.com/r/NixOS/comments/16bwuvu/can_you_calculate_zram_size_dependent_on_machine/)
[^2]: https://access.redhat.com/solutions/1301563
[^4]: https://www.shellcheck.net/
[^5]: https://github.com/dntco43u/s6h7k8rv/blob/main/cleanup_disk.sh
[^6]: https://github.com/dntco43u/s6h7k8rv/blob/main/dpasswd.sh
[^7]: https://github.com/dntco43u/s6h7k8rv/blob/main/notify_df.sh
[^8]: https://grafana.com/grafana/dashboards/1860-node-exporter-full
[^9]: https://rockylinux.org/legal/licensing
