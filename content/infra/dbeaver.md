---
date: 2025-12-13
title: dbeaver
description: dbeaver 구성
tags:
  - dbeaver
  - ide
categories:
  - infra
---

![image](/images/dbeaver-logo.webp#center)

## License
상업적 이용 제한 없음
- dbeaver-ce: Apaache v2 [^2]
- D2Coding: OFL [^3]

## Troubleshooting
{{% alert color=warning %}}
> Error resolving dependencies Maven artifact 'maven:/com.oracle.database.xml:xmlparserv2:RELEASE' not found

oracle driver 설치 실패. dbeaver 관리자 권한으로 실행
{{% /alert %}}

{{% alert color=warning %}}
> Error : 1031, Position : 0, SQL = ALTER SESSION SET "_optimizer_push_pred_cost_based" = FALSE, Original SQL = ALTER SESSION SET "_optimizer_push_pred_cost_based" = FALSE, Error Message = ORA-01031: 권한이 불충분합니다

dbeaver ui에서 Edit Connection -> Connection settings -> Oracle properties 탭 -> Use metadata queries optimizer 체크 해제
{{% /alert %}}

{{% alert color=warning %}}
> Invalid preference category path: org.eclipse.debug.ui.ConsolePreferencePage (bundle: org.eclipse.ui.console, page: org.eclipse.ui.internal.console.ansi.preferences.AnsiConsolePreferencePage)

dbeaver v22.2.3 이후부터 발생함. 재설치나 수동 리셋해도 반복됨<br>
오류 없는 마지막 구버전 [dbeaver-ce-22.2.2-x86_64-setup.exe](https://dbeaver.io/files/22.2.2/dbeaver-ce-22.2.2-x86_64-setup.exe)<br>
dbeaver v25.3.0 에러 메시지 로그 외에는 사용에 지장 없음 (2025-12)
{{% /alert %}}

{{% alert color=warning %}}
> postgresql ERROR: operator is not unique: "char" || text

posggresql v15 이상에서 발생함. dbeaver v22.3.1에서 수정
{{% /alert %}}

[^2]: https://dbeaver.io/product/dbeaver_license.txt
[^3]: https://github.com/naver/d2codingfont
