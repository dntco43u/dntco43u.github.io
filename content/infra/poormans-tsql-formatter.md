---
date: 2025-12-13
title: PoorMansTSqlFormatter
description: PoorMansTSqlFormatter 구성
tags:
  - code-formatter
  - dbeaver
  - ide
categories:
  - infra
---

![image](/images/sh-logo.webp#center)

## windows 구성

### 개요
- 들여쓰기 탭 대신 공백 2자리
- 키워드 표준화
- 키워드 대문자
- 탭 크기 0
- 콤마 뒤 열 공백 추가
- 파싱 에러 허용
- BETWEEN 키워드 행 확장 거부
- IN 키워드 행 확장 거부

### dbeaver
| Options       |                                                                                                           |
|---------------|-----------------------------------------------------------------------------------------------------------|
| command line  | "c:\Users\dev\AppData\Roaming\SqlFormatter\SqlFormatter.exe" /is:"  " /sk- /uk /st:0 /sac /ae /ebc- /eil- |
| Formatter     | External formatter                                                                                        |
| Use temp file | Uncheck [^1]                                                                                              |

```sql
SELECT *
FROM TABLE1 t
WHERE a > 100
  AND b BETWEEN 12 AND 45;

SELECT t.*
  , j1.x
  , j2.y
FROM TABLE1 t
JOIN JT1 j1 ON j1.a = t.a
LEFT OUTER JOIN JT2 j2 ON j2.a = t.a
  AND j2.b = j1.b
WHERE t.xxx IS NOT NULL;

DELETE
FROM TABLE1
WHERE a = 1;

UPDATE TABLE1
SET a = 2
WHERE a = 1

SELECT table1.id
  , table2.number
  , SUM(table1.amount)
FROM table1
INNER JOIN table2 ON table1.id = table2.table1_id
WHERE table1.id IN (
    SELECT table1_id
    FROM table3
    WHERE table3.name = 'Foo Bar'
      AND table3.type = 'unknown_type'
    )
GROUP BY table1.id
  , table2.number
ORDER BY table1.id;
```

### command options
| Options | Remarks                                                                                |
|---------|----------------------------------------------------------------------------------------|
| is      | indentString (default: \t)                                                             |
| st      | spacesPerTab (default: 4)                                                              |
| mw      | maxLineWidth (default: 999)                                                            |
| sb      | statementBreaks (default: 2)                                                           |
| cb      | clauseBreaks (default: 1)                                                              |
| tc      | trailingCommas (default: false)                                                        |
| sac     | spaceAfterExpandedComma (default: false)                                               |
| ebc     | expandBetweenConditions (default: true)                                                |
| ebe     | expandBooleanExpressions (default: true)                                               |
| ecs     | expandCaseStatements (default: true)                                                   |
| ecl     | expandCommaLists (default: true)                                                       |
| eil     | expandInLists (default: true)                                                          |
| uk      | uppercaseKeywords (default: true)                                                      |
| sk      | standardizeKeywords (default: false)                                                   |
| ae      | allowParsingErrors (default: false)                                                    |
| e       | extensions (default: sql)                                                              |
| r       | recursive (default: false)                                                             |
| b       | backups (default: true)                                                                |
| b       | outputFileOrFolder (default: none; if set, overrides the backup option)                |
| l       | languageCode (default: current if supported or EN; valid values include EN, FR and ES) |

## License
상업적 이용 제한 없음
- AGPL v3.0.1 [^2]

## Troubleshooting
{{% alert color=warning %}}
> Unrecognized arguments found!

dbeaver ui에서 SQL Format -> Use temp file 옵션 체크 해제
{{% /alert %}}

## References
- https://github.com/TaoK/PoorMansTSqlFormatter

[^1]: https://github.com/TaoK/PoorMansTSqlFormatter/issues/285
[^2]: https://github.com/TaoK/PoorMansTSqlFormatter?tab=AGPL-3.0-1-ov-file
