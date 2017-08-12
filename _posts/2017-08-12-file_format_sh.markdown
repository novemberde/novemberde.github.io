---
title: "/bin/bash^M: bad interpreter: No such file or directory"
category: bash
tags: bin/bash, bash, bad interpreter in windows, sh fileformat in windows
---
 
Windows에서 shell script를 작성하여 배포하였을 경우에 다음과 같은 에러를 만날 수 있다.

```bash
$ bash SomeScript.sh
/bin/bash^M: bad interpreter: No such file or directory
```

이것은 CRLF를 Windows에서 기본으로 사용하였기 때문에 나타난다.
리눅스 기반은 LF를 기반으로 하기 때문에 newline문자가 달라지기 때문이다.
VS code를 사용한다면 오른쪽 맨 밑에 CRLF를 LF를 수정하면 해결된다.

만일 다른 에디터를 사용한다면 설정메뉴에서 바꾸면 될 것이다.

---
### CRLF와 LF란
---

CR과 LF의 뜻을 알아보자.

- CR: Carriage Return
- LF: Line Feed

CR은 커서의 위치를 가장 앞으로 옮기는 동작이며, LF는 커서의 위치를 한칸 내리는 것을 의미한다.

CRLF와 LF의 차이는 \r\n과 \n의 차이이다.

