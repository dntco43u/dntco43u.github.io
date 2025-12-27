---
date: 2025-12-14
title: doublecmd
description: Double Commander 구성
tags:
  - doublecmd
categories:
  - infra
---

![image](/images/doublecmd-logo.webp#center)

![image](/images/doublecmd-1.webp)

## windows 구성

### 개요
- Mdir 참조 확장자별 색상
- 기타 편의 구성
- D2Coding 의존

### 다운로드
[doublecmd-settings.zip](../doublecmd-settings.zip)<br>
[D2Coding-Ver1.3.2-20180524.zip](https://github.com/naver/d2codingfont/releases/download/VER1.3.2/D2Coding-Ver1.3.2-20180524.zip)

| Filename       | Remarks                 |
|----------------|-------------------------|
| sevenzip.ini   | 7z 플러그인             |
| colors.json    | 모든 색상 설정          |
| ignorelist.txt | 파일/폴더 무시          |
| doublecmd.xml  | 모든 주요 프로그램 설정 |
| extassoc.xml   | 파일 확장자 연관 설정   |

### gitbash로 설치
```sh
curl https://dntco43u.github.io/infra/doublecmd-settings.zip -o $USERPROFILE/Downloads/doublecmd-settings.zip && \
7z x -y -o"$APPDATA/doublecmd" $USERPROFILE/Downloads/doublecmd-settings.zip && \
rm $USERPROFILE/Downloads/doublecmd-settings.zip
```

## License
상업적 이용 제한 없음
- Double Commander: GNU GPL v2 [^1]
- D2Coding: OFL [^2]

## References
- https://github.com/doublecmd/doublecmd
- https://doublecmd.github.io/doc/en/configuration.html

[^1]: https://github.com/doublecmd/doublecmd?tab=GPL-2.0-1-ov-file
[^2]: https://github.com/naver/d2codingfont
