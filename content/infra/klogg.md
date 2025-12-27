---
date: 2025-12-14
title: klogg
description: KLOGG 구성
tags:
  - klogg
categories:
  - infra
---

![image](/images/klogg-logo.webp#center)

![image](/images/klogg-1.webp)

## windows 구성

### 개요
- 레벨별 색상 구성

  ```
  d{4}[\\/-]\\d{2}[\\/-]\\d{2}\\s\\d{2}:\\d{2}:\\d{2}.*TRACE|DEBUG
  d{4}[\\/-]\\d{2}[\\/-]\\d{2}\\s\\d{2}:\\d{2}:\\d{2}.*INFO|NOTICE
  d{4}[\\/-]\\d{2}[\\/-]\\d{2}\\s\\d{2}:\\d{2}:\\d{2}.*WARN
  d{4}[\\/-]\\d{2}[\\/-]\\d{2}\\s\\d{2}:\\d{2}:\\d{2}.*ERROR|SEVERE|CRITICAL
  ```
- D2Coding 의존

### 다운로드
[klogg-settings.zip](../klogg-settings.zip)<br>
[D2Coding-Ver1.3.2-20180524.zip](https://github.com/naver/d2codingfont/releases/download/VER1.3.2/D2Coding-Ver1.3.2-20180524.zip)

| Filename   | Remarks |
|------------|---------|
| klogg.conf | 구성    |

### gitbash로 설치
```sh
curl https://dntco43u.github.io/infra/klogg-settings.zip -o $USERPROFILE/Downloads/klogg-settings.zip && \
7z x -y -o"$USERPROFILE/Downloads" $USERPROFILE/Downloads/klogg-settings.zip && \
mv $USERPROFILE/Downloads/klogg.conf "$PROGRAMFILES/klogg" && \
rm $USERPROFILE/Downloads/klogg-settings.zip
```

## License
상업적 이용 제한 없음
- klogg: GNU GPL v1 [^1]
- D2Coding: OFL [^2]

## References
- https://github.com/variar/klogg

[^1]: https://github.com/variar/klogg?tab=License-1-ov-file
[^2]: https://github.com/naver/d2codingfont
