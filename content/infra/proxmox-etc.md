---
date: 2025-12-09
title: proxmox 기타 구성
description: proxmox 기타 구성 메모
tags:
  - proxmox
categories:
  - infra
---

![image](/images/proxmox-logo.webp#center)

## proxmox 기타

### local-lvm 삭제
```sh
lvremove /dev/pve/data && \
lvresize -l +100%FREE /dev/pve/root && \
resize2fs /dev/mapper/pve-root
```

### node 이름 변경
```sh
TARGET_HOST=xxxxxxxx && \
sed -i "s/$TARGET_HOST/p6dhywxp/" /etc/hostname && \
sed -i "s/$TARGET_HOST/p6dhywxp/" /etc/hosts && \
sed -i "s/$TARGET_HOST/p6dhywxp/" /etc/postfix/main.cf && \
mv /var/lib/rrdcached/db/pve2-node/$TARGET_HOST /var/lib/rrdcached/db/pve2-node/p6dhywxp && \
mv /var/lib/rrdcached/db/pve2-storage/$TARGET_HOST /var/lib/rrdcached/db/pve2-storage/p6dhywxp && \
rm -rf /var/lib/rrdcached/db/pve2-{node,storage}/$TARGET_HOST && \
ls /var/lib/rrdcached/db/pve2-{node,storage} && \
systemctl stop pve-cluster && systemctl stop pvestatd && \
systemctl restart pve-cluster && systemctl restart pvestatd
```

### no-subscription
web--ui에서 node -> hostname -> repositories -> disable enterprise/enable no subscription
```sh
vi /etc/apt/sources.list.d/pve-enterprise.list
```
```sh
# deb https://enterprise.proxmox.com/debian... bookworm eve-enterprise
```

```sh
vi /etc/apt/sources.list
```
```sh
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
```

```sh
vi /etc/apt/sources.list.d/ceph.list
```
```sh
# deb https://enterprise.proxmox.com/debian... bookworm enterprise
deb http://download.proxmox.com/debian/ce... bookworm no-subscription
```

### 구독 팝업 삭제
```sh
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js{,.bak} && \
sed -i "/^ *checked_command:/,/^ *[a-zA-Z_]+:/{ s/status\.toLowerCase() !== 'active'/\0 \&\& false/ }" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
```

### microcode 패치
```sh
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/microcode.sh)"
```

### repo, subscription nag 비활성화
```sh
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
```

### use tablet for pointer 비활성화
```sh
for vm in $(qm list | awk '{print $1}' | grep [0-9]); do qm set $vm --tablet 0; done
```

## 절전 공통

### powertop
```sh
apt update -y && \
apt install powertop -y && \
powertop --auto-tune && \
systemctl enable powertop && systemctl restart powertop
```

### aspm
```sh
dmesg | grep ASPM && \
lspci -vv | awk '/ASPM/{print $0}' RS= | grep --color -P '(^[a-z0-9:.]+|ASPM )'
```

### aspm.py [^9]
재시작 시 aspm 활성화
```sh
cd /usr/local/bin && curl -OL https://raw.githubusercontent.com/0x666690/ASPM/main/aspm.py
```
```sh
crontab -e
```
```sh
...
@reboot /usr/bin/python3 /usr/local/bin/aspm.py
...
```
```sh
systemctl restart crond
```

## intel cpu
cup 세대에 따라 절전 구성 지원 범위가 다르므로 테스트할 것<br>
| CPU          | hwp/epb   | intel_pstate   | scaling_governor   |
| ------------ | --------- | -------------- | ------------------ |
| j5005 [^5]   | ×         | ✓              | ✓                  |

### hwp/epb
구성 예 [^6]
```sh
x86_energy_perf_policy --hwp-min 8 --hwp-max 10 --turbo-enable 0;
x86_energy_perf_policy --hwp-epp 255 --turbo-enable 0;
x86_energy_perf_policy --hwp-epp 0 --turbo-enable 1;
x86_energy_perf_policy --hwp-min 8 --hwp-max 28 --turbo-enable 1;
```

구성 확인
```sh
cat /sys/devices/system/cpu/cpu*/power/energy_perf_bias && \
cat /sys/devices/system/cpu/cpu*/cpufreq/energy_performance_preference
```

### intel_pstate [^3]
구성 예
```sh
echo 70 | tee /sys/devices/system/cpu/intel_pstate/max_perf_pct;
echo 100 | tee /sys/devices/system/cpu/intel_pstate/max_perf_pct;
```

재시작해도 구성 유지하려면 커널 구성에 반영
```sh
vi /etc/default/grub
```
```ini
...
GRUB_CMDLINE_LINUX="intel_pstate=active pcie_aspm=force pcie_aspm.policy=powersupersave"
...
```
```sh
update-grub && update-initramfs -u
```

구성 확인
```sh
cat /sys/devices/system/cpu/intel_pstate/no_turbo && \
cat /sys/devices/system/cpu/intel_pstate/status && \
cat /sys/devices/system/cpu/intel_pstate/num_pstates && \
cat /sys/devices/system/cpu/intel_pstate/turbo_pct && \
cat /sys/devices/system/cpu/intel_pstate/max_perf_pct && \
cat /sys/devices/system/cpu/intel_pstate/min_perf_pct
```

### governor
구성 예
```sh
echo "powersave" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor;
echo "performance" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor;
echo 800000 | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_min_freq;
echo 2800000 | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_max_freq;
echo 2800000 | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_setspeed;
```

cpufrequtils [^4]
```sh
apt update -y && apt install -y cpufrequtils && \
cat << 'EOF' > /etc/default/cpufrequtils
GOVERNOR="powersave"
EOF
```
```sh
cat /etc/default/cpufrequtils && \
cpufreq-info
```

구성 확인
```sh
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor && \
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_min_freq && \
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_max_freq && \
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_setspeed
```

### iommu (VT-d)
```sh
dmesg | grep -e DMAR -e IOMMU && \
vi /etc/default/grub
```
```ini
...
GRUB_CMDLINE_LINUX="intel_iommu=on"
...
```
```sh
update-grub && reboot
```

## disk

### qcow2 이미지 결합
```sh
qemu-img info /var/lib/vz/images/100/vm-100-disk-0.qcow2 && \
qm config 100 && \
pvesm status && \
qm set 100 -scsi0 /var/lib/vz/images/100/vm-100-disk-0.qcow2
```

### disk passthrough
```sh
lsblk |awk 'NR==1{print $0" DEVICE-ID(S)"}NR>1{dev=$1;printf $0" ";system("find /dev/disk/by-id -lname \"*"dev"\" -printf \" %p\"");print "";}'|grep -v -E 'part|lvm' && \
qm config 100 | egrep --color -i "^(sata|scsi)"
```
```sh
qm set 100 -scsi1 /dev/disk/by-id/ata-WDC_WD******-*******_**-********
```

### uuid로 디스크 식별자 고정 [^1]
```sh
"$(readlink -f /dev/disk/by-uuid/f*******-****-****-****-************)"
```

### 기타
모든 SMART와 비SMART, 쓰기 캐시, apm 상태
```sh
smartctl -x /dev/sda && \
hdparm -W /dev/sda && \
hdparm -B /dev/sda
```

hdd 절전 모드 헤드 파킹 비활성화
```sh
hdparm -C /dev/disk/by-uuid/f*******-****-****-****-************ && \
hdparm -S 0 /dev/disk/by-uuid/f*******-****-****-****-************
```

ssd 파티션 정렬 상태
```sh
apt install -y parted && \
parted /dev/nvme0n1
```

## nic [^7]

### nic 이름 변경
```sh
ip addr
```
```sh
vi /etc/network/interfaces
```

Proxmox naming convention
| Name     | Remarks                                |
|----------|----------------------------------------|
| eno1     | 최초의 온보드 NIC                      |
| enp3s0f1 | PCI 버스 3, 슬롯 0에 있는 NIC의 기능 1 |

```sh
service networking restart
```

### wakeonlan
```sh
ethtool -s enp2s0 wol g && \
ethtool enp2s0 | grep Wake-on && \
vi /etc/network/interfaces
```
```
iface enp2s0 inet manual
        post-up /usr/sbin/ethtool -s enp2s0 wol g
```

### aspm
```sh
cat /sys/class/net/enp2s0/device/link/* && \
cat /sys/module/pcie_aspm/parameters/policy
```

### eee 비활성화 [^8]
realtek nic가 간헐적으로 끊어질 때 시도할 것
```sh
ethtool --show-eee enp2s0 && \
ethtool --set-eee enp2s0 eee off tx-lpi off
```

### r8169 -> r8168
```sh
apt update -y && \
apt install pve-headers build-essential -y && \
curl -OL https://github.com/sbwml/package_kernel_r8168/releases/download/8.053.00/r8168-8.053.00.tar.bz2 && \
tar -xvf r8168-8.053.00.tar.bz2 && \
cd r8168-8.053.00 && \
chmod 700 autorun.sh && \
./autorun.sh && \
echo "blacklist r8169" >> /etc/modprobe.d/pve-blacklist.conf && \
cat /etc/modprobe.d/pve-blacklist.conf && \
lspci -vv && \
lsmod | grep "r816" && \
systemctl restart networking && \
ip addr && \
cd .. && rm r8168-8.053.00.tar.bz2 && rm -rf /r8168-8.053.00
```

```sh
echo 0 | tee /sys/class/net/enp2s0/device/link/l1_1_aspm && \
echo 0 | tee /sys/class/net/enp2s0/device/link/l1_2_aspm && \
echo 1 | tee /sys/class/net/enp2s0/device/link/l1_aspm
```

### r8168 -> r8169
```sh
apt update -y && \
apt install pve-kernel pve-headers build-essential -y && \
modprobe -r -v r8168 && \
modprobe -v r8169 && \
echo "blacklist r8168" >> /etc/modprobe.d/pve-blacklist.conf && \
cat /etc/modprobe.d/pve-blacklist.conf && \
update-grub && \
update-initramfs -u && \
reboot
```

```sh
lspci -vv && \
insmod /lib/modules/6.8.12-1-pve/kernel/drivers/net/phy/realtek.ko && \
insmod /lib/modules/6.8.12-1-pve/kernel/drivers/net/ethernet/realtek/r8169.ko && \
find / -name *r8168* -exec rm -rf {} \;
```

## License
상업적 이용 제한 없음
- pve-no-subscription | AGPL v3 [^2]

## Troubleshooting
{{% alert color=warning %}}
> web-ui 501

```sh
systemctl restart pveproxy
```
{{% /alert %}}

{{% alert color=warning %}}
> ```sh
> dmesg
> ```
> [ 1075.771449] r8168 0000:02:00.0: Unable to change power state from unknown to D0, device inaccessible<br>
> [ 1075.772728] r8168 0000:02:00.0: Unable to change power state from D3cold to D0, device inaccessible

r8111gn은 r8168 + l1_aspm으로 구성하면 정상 작동하지만 l1_1_aspm이나 l1_2_aspm을 추가하면 먹통 됨<br>
r8168 드라이버가 aspm 문제를 해결할 때까지 r8169 사용할 것
{{% /alert %}}

## References
- https://manpages.debian.org/experimental/linux-cpupower/x86_energy_perf_policy.8.en.html
- https://manpages.debian.org/bookworm/smartmontools/smartctl.8.en.html
- https://proxmox.com/en/products/proxmox-virtual-environment/comparison
- https://z8.re/blog/aspm.html
- https://www.reddit.com/r/Proxmox/comments/tgojp1/removing_proxmox_subscription_notice/?tl=ko
- https://www.reddit.com/r/linux4noobs/comments/1fzupnf/why_linux_by_default_install_r8169_instead_of/
- https://www.reddit.com/r/linux4noobs/comments/168ca7f/r8168_driver_installed_instead_of_r8169_on_ubuntu/
- https://www.reddit.com/r/Proxmox/comments/150stgh/proxmox_8_rtl8169_nic_dell_micro_formfactors_in/
- https://medium.com/@pattapongj/how-to-fix-network-issues-after-upgrading-proxmox-from-7-to-8-and-encountering-the-r8169-error-d2e322cc26ed
- https://rovingclimber.com/2023/07/28/proxmox-8-debian-12-with-realtek-rtl8111h-nic/
- https://rovingclimber.com/category/home-server/
- https://mattgadient.com/7-watts-idle-on-intel-12th-13th-gen-the-foundation-for-building-a-low-power-server-nas/

[^1]: https://forum.proxmox.com/threads/system-disk-number-changed-when-i-add-new-drive.98338/
[^2]: https://forum.proxmox.com/threads/does-proxmox-still-offer-a-fully-free-version.146066/
[^3]: https://www.kernel.org/doc/html/latest/admin-guide/pm/intel_pstate.html
[^5]: intel_pstate=per_cpu_perf_limits 공통 cpufreq에 접근 가능하나 passive로 변경됨
[^6]: --hwp-min, --hwp-max, --hwp-desired의 경우 값 옵션은 100MHz 단위. 12는 1200MHz를 의미
[^4]: gorvernor와 같은 설정을 참조
[^7]: 구형 realtek nic들과 linux 사이에 해결할 수 없는 문제가 많음. r8168과 r8169 드라이버는 의견도 갈리고 커널과 드라이버 버전 경우의 수도 많아서 양쪽을 테스트할 것
[^8]: https://en.wikipedia.org/wiki/Energy-Efficient_Ethernet
[^9]: https://github.com/0x666690/ASPM
