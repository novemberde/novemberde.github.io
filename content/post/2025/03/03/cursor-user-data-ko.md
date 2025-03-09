---
title: "언어별로 여러 Cursor 또는 VSCode를 사용하는 방법"
tags: ["cursor", "vscode", "editor", "method", "ide", "user-data-dir", "코드편집기]
date: "2025-03-03T00:30:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

한동안 Cursor를 사용했고 이것이 훌륭한 에디터라고 생각한다. 하지만 여러 프로그래밍 언어가 필요한 프로젝트에서 작업할 때, 종종 에디터가 무거워지는 경우가 있었다. 이럴 때 생산성이 떨어져서 각 언어마다 다른 에디터 설정을 사용하는 방법을 찾고 있었다.

# Profile을 이용하는 방법

Cursor에서는 Profile을 이용해서 각 언어별로 다른 설정을 사용할 수 있다. 이 방법은 각 언어별로 다른 설정을 사용하는 것이 아니라, 각 언어별로 다른 프로필을 사용하는 것이다.

`cmd+shift+p` 를 눌러서 `>Profiles: New Profile...` 를 선택하면 profile을 생성할 수 있다:

생성했다면 다시 `cmd+shift+p` 를 눌러서 `>Profiles: Switch Profile...` 을 선택하면 생성한 profile을 선택할 수 있다:

그럼 해당 프로젝트에서 독립된 프로필로 환경을 설정할 수 있다.

# user-data-dir을 이용하는 방법

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

이 방법은 언어별로 여러 에디터 구성을 사용하는 데 좋다. VSCode와도 호환된다.

하지만 이것을 더 발전시키고 싶었다. 단순히 다른 구성을 갖는 것이 아니라, CLI에서 바로 다른 에디터 인스턴스를 사용하고 싶었다.

Zsh를 사용하기 때문에 `.zshrc` 파일에 다음과 같이 alias를 설정했다:

```bash
alias pcursor="cursor --user-data-dir=~/.cursor-python"
alias gcursor="cursor --user-data-dir=~/.cursor-go"
alias kcursor="cursor --user-data-dir=~/.cursor-kotlin"
```

이제 각 언어별로 Cursor를 열 수 있다:

```bash
cd YOUR_DIRECTORY && pcursor
cd YOUR_DIRECTORY && gcursor
cd YOUR_DIRECTORY && kcursor
```

각 언어별로 alias를 통해 여러 Cursor 또는 VSCode 인스턴스를 사용할 수 있게 한다.

여러 언어를 위한 많은 플러그인을 사용할 때 에디터 성능이 저하되는 경우를 방지할 수 있고, 별도의 구성을 유지하는 방식은 에디터를 크게 느려지지 않게 하기 때문에 생산성을 크게 향상시킬 수 있다. 도움이 되길 바란다.
