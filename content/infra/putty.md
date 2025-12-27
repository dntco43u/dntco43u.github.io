---
date: 2025-12-14
title: putty
description: PuTTY 구성
tags:
  - putty
categories:
  - infra
---

![image](/images/putty-logo.webp#center)

![image](/images/putty-1.webp)

## windows 구성

### 개요
- 색상 [^1]
- 접속 정보를 제외하고 구성된 dev@hostaddr 세션
- D2Coding 의존

### 다운로드
[putty-sessions.zip](../putty-sessions.zip)<br>
[D2Coding-Ver1.3.2-20180524.zip](https://github.com/naver/d2codingfont/releases/download/VER1.3.2/D2Coding-Ver1.3.2-20180524.zip)

| Filename           | Remarks                   |
|--------------------|---------------------------|
| putty-sessions.reg | 세션 정보 레지스트리      |

### gitbash로 설치
```sh
curl https://dntco43u.github.io/infra/putty-sessions.zip -o $USERPROFILE/Downloads/putty-sessions.zip && \
7z x -y -o"$USERPROFILE/Downloads" $USERPROFILE/Downloads/putty-sessions.zip && \
reg import $USERPROFILE/Downloads/putty-sessions.reg && \
find $USERPROFILE/Downloads/ -type f -name putty-sessions.* -exec rm {} \;
```

## License
상업적 이용 제한 없음
- putty: MIT [^1]
- D2Coding: OFL [^2]

[^1]: https://stackoverflow.com/questions/49075439/putty-monokai-color-scheme-manually
[^2]: https://www.chiark.greenend.org.uk/~sgtatham/putty/licence.html
[^3]: https://github.com/naver/d2codingfont
