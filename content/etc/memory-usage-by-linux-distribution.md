---
date: 2025-12-07
title: 배포판 별 메모리 사용량
description: linux 배포판 별 실제 가용 메모리를 확인
tags:
  - rhel
  - debian
  - benchmark
categories:
  - etc
---

![image](/images/linux-logo.webp#center)

## 가용 메모리 비교
설치 후 초기 상태에서 비교
```sh
free -h
```
- | ubuntu 22.04 lts      |             |               |            |                   |                    |
  |-----------------------|-------------|---------------|------------|-------------------|--------------------|
  | total                 | used        | free          | shared     | buff/cache        | available          |
  | Mem:             833  | 239         | 216           | 3          | 377               | 449                |
  | Swap:              0  | 0           | 0             |            |                   |                    |

- | oracle linux          |             |               |            |                   |                    |
  |-----------------------|-------------|---------------|------------|-------------------|--------------------|
  | total                 | used        | free          | shared     | buff/cache        | available          |
  | Mem:          682Mi   | 237Mi       | 144Mi         | 4.0Mi      | 300Mi             | 335Mi              |
  | Swap:         1.3Gi   | 1.0Mi       | 1.3Gi         |            |                   |                    |

- | rockylinux (migrated) |             |               |            |                   |                    |
  | --------------------- | ----------- | ------------- | ---------- | ----------------- | ------------------ |
  | total                 | used        | free          | shared     | buff/cache        | available          |
  | Mem:          959Mi   | 213Mi       | 477Mi         | 6.0Mi      | 267Mi             | 603Mi              |
  | Swap:            0B   | 0B          | 0B            |            |                   |                    |

- | debian 11 bullseye    |             |               |            |                   |                    |
  |-----------------------|-------------|---------------|------------|-------------------|--------------------|
  | total                 | used        | free          | shared     | buff/cache        | available          |
  | Mem:             863  | 133         | 405           | 0          | 323               | 600                |
  | Swap:              0  | 0           | 0             |            |                   |                    |

- | ubuntu minimal        |             |               |            |                   |                    |
  |-----------------------|-------------|---------------|------------|-------------------|--------------------|
  | total                 | used        | free          | shared     | buff/cache        | available          |
  | Mem:          966Mi   | 121Mi       | 456Mi         | 1.0Mi      | 388Mi             | 706Mi              |
  | Swap:            0B   | 0B          | 0B            |            |                   |                    |

{{% alert title="Note" color="primary" %}}
rockylinux는 debian 수준으로 가볍다
{{% /alert %}}
