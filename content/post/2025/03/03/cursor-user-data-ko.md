---
title: "언어별로 여러 Cursor 또는 VSCode IDE 사용하는 방법"
tags: ["cursor", "vscode", "IDE", "method"]
date: "2025-03-03T00:30:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

한동안 Cursor를 사용했고 이것이 훌륭한 IDE라고 생각한다. 하지만 여러 프로그래밍 언어가 필요한 프로젝트에서 작업할 때, 종종 IDE가 무거워지는 경우가 있었다. 이럴 때 생산성이 떨어져서 각 언어마다 다른 IDE 설정을 사용하는 방법을 찾고 있었다.

`--user-data-dir` 플래그를 사용하면 이것이 가능하다. 이 플래그는 각 언어 구성에 대해 다른 user-data directory를 지정할 수 있게 한다.

예를 들어, 다른 프로그래밍 언어에 Cursor를 사용하고 싶다면 다음 명령어를 실행할 수 있다:

```bash
# Python 사용
cursor --user-data-dir=~/.cursor-python

# Go 사용
cursor --user-data-dir=~/.cursor-go

# Kotlin 사용
cursor --user-data-dir=~/.cursor-kotlin
```

이 방법은 언어별로 여러 IDE 구성을 사용하는 데 좋다. VSCode와도 호환된다.

하지만 이것을 더 발전시키고 싶었다. 단순히 다른 구성을 갖는 것이 아니라, CLI에서 바로 다른 IDE 인스턴스를 사용하고 싶었다.

Zsh를 사용하기 때문에 `.zshrc` 파일에 다음과 같이 alias를 설정했다:

```bash
alias pcursor="cursor --user-data-dir=~/.cursor-python"
alias gcursor="cursor --user-data-dir=~/.cursor-go"
alias kcursor="cursor --user-data-dir=~/.cursor-kotlin"
```

이제 각 언어별로 Cursor를 열 수 있다:

```bash
pcursor YOUR_DIRECTORY
gcursor YOUR_DIRECTORY
kcursor YOUR_DIRECTORY
```

각 언어별로 alias를 통해 여러 Cursor 또는 VSCode 인스턴스를 사용할 수 있게 한다.

여러 언어를 위한 많은 플러그인을 사용할 때 IDE 성능이 저하되는 경우를 방지할 수 있고, 별도의 구성을 유지하는 방식은 IDE를 크게 느려지지 않게 하기 때문에 생산성을 크게 향상시킬 수 있다. 도움이 되길 바란다.
