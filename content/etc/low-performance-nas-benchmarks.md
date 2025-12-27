---
date: 2025-12-07
title: 구형 nas 벤치마크
description: 蜗牛星际 snail interstellar 타오나스, lenovo v330-15igm 등 저성능 서버 yabs.sh 벤치마크
tags:
  - rhel
  - benchmark
categories:
  - etc
---

![image](/images/yabs-logo.webp#center)

## 벤치마크

### geekbench4 [^1]
```sh
yabs.sh
```
```sh
#!/bin/bash
# geekbench 벤치마크

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

curl -sL https://raw.githubusercontent.com/masonr/yet-another-bench-script/master/yabs.sh \
  | bash -s -- -bi4 > "$log_file"
{ grep -oE "completed in.*" "$log_file" \
    | sed -E 's/(completed in.)(.*)/Elapsed time: \2/g'
  grep -E "Host\s*:" "$log_file" \
    | sed -E 's/(Host)(\s*)(:.*)/\1\3/g'
  grep -E "Distro\s*:" "$log_file" \
    | sed -E 's/(Distro)(\s*)(:.*)/OS\3/g'
  cpu="$(grep -E "Processor\s*:" "$log_file" \
    | sed -E 's/(Processor)(\s*)(:.*)/CPU\3/g')"
  ram="$(grep -E "RAM\s:*" "$log_file" \
    | sed -E 's/(RAM)(\s*)(:.*)/\1\3/g')"
  disk="$(grep -E "Disk\s*:" "$log_file" \
    | sed -E 's/(Disk)(\s*)(:.*)/\1\3/g')"
  echo "$cpu, $ram, $disk"

  single="$(grep -E "Single Core" "$log_file" \
    | sed -E 's/(Single Core)(\s*\| )([0-9]*)(\s*)/\1: \3/g')"
  multi="$(grep -E "Multi Core" "$log_file" \
    | sed -E 's/(Multi Core)(\s*\| )([0-9]*)(\s*)/\1: \3/g')"
  echo "$single, $multi"

  grep "Read" "$log_file" \
    | tail -n 1 \
    | awk '{ print "1M "$1": "$7" "$8",  IOPS: "$9 }' \
    | sed -E 's/\(|\)//g'
  grep "Write" "$log_file" \
    | tail -n 1 \
    | awk '{ print "1M "$1": "$7" "$8",  IOPS: "$9 }' \
    | sed -E 's/\(|\)//g'
  grep "Read" "$log_file" \
    | head -n 1 \
    | awk '{ print "4K "$1": "$3" "$4",  IOPS: "$5 }' \
    | sed -E 's/\(|\)//g'
  grep "Write" "$log_file" \
    | head -n 1 \
    | awk '{ print "4K "$1": "$3" "$4",  IOPS: "$5 }' \
    | sed -E 's/\(|\)//g'
  grep "Full Test" "$log_file" \
    | sed -E 's/(Full Test)(\s*\| )(.*)/\3/g'
  rm /home/dev/geekbench_claim.url
} > "$msg_file"
send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
rm "$msg_file"
```

-   | J1900 + 850EVO |             |            |       |
    |----------------|-------------|------------|-------|
    | Single Core    | 240         | Multi Core | 705   |
    | 1M Read        | 130.57 MB/s | IOPS       | 127   |
    | 1M Write       | 139.27 MB/s | IOPS       | 136   |
    | 4K Read        | 94.11 MB/s  | IOPS       | 23.5k |
    | 4K Write       | 94.36 MB/s  | IOPS       | 23.5k |


-   | J5005 + PM871B |             |            |       |
    |----------------|-------------|------------|-------|
    | Single Core    | 494         | Multi Core | 1588  |
    | 1M Read        | 118.12 MB/s | IOPS       | 115   |
    | 1M Write       | 125.98 MB/s | IOPS       | 123   |
    | 4K Read        | 93.56 MB/s  | IOPS       | 23.3k |
    | 4K Write       | 93.81 MB/s  | IOPS       | 23.4k |


-   | J5005 + PM9A1 |             |            |       |
    |---------------|-------------|------------|-------|
    | Single Core   | 494         | Multi Core | 1578  |
    | 1M Read       | 321.23 MB/s | IOPS       | 313   |
    | 1M Write      | 342.62 MB/s | IOPS       | 334   |
    | 4K Read       | 230.73 MB/s | IOPS       | 57.6k |
    | 4K Write      | 231.34 MB/s | IOPS       | 57.8k |


### geekbench5

- J5005 + PM9A1
  ![image](/images/low-performance-nas-benchmarks-1.webp)

[^1]: https://github.com/dntco43u/s6h7k8rv/blob/main/yabs.sh
