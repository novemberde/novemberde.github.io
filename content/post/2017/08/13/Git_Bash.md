---
title: VS code에서 git bash 사용하기
tags: [bash, git, vscode, gitbash]
date: "2017-08-13T11:30:03+00:00"
aliases: ["/bash/2017/08/13/Git_Bash.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

Windows에서 Power shell이나 CMD로 작업하면 linux 명령어를 사용하기 불편하다.

그렇기 때문에 종종 git bash를 사용했는데 매번 git bash를 켜서 작업하는 것도 귀찮았다.

Visual Studio Code에서 git bash를 기본 터미널로 설정하면 편하게 작업할 수 있을 것이다.

방법은 설정에서 사용자 정의 부분을 아래와 같이 덮어씌우면 된다.

```json
{
  "terminal.integrated.shell.windows": "C:\\Program Files\\Git\\bin\\bash.exe"
}
```

이제 linux 명령어를 편하게 사용하면서 작업하자.

---
### Reference
---

- [https://code.visualstudio.com/docs/editor/integrated-terminal](https://code.visualstudio.com/docs/editor/integrated-terminal)