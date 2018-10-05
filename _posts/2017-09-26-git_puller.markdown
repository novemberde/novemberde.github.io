---
title: Git pull all sub repository of certain directory.
category: git
tags: git, pull, subdirectory, directory, sub, renew, 갱신, puller, list
---
## Summary
---
여러 프로젝트를 동시에 진행하다보니 여러 repository를 한번에 갱신할 필요성이 생겼다. 일일이 하나씩 들어가서 git pull을 하려니 은근 귀찮았다.
그렇기 때문에 특정 디렉터리에 있는 all repositories(sub directories)에 대해서 git을 갱신해주는 코드를 개발하였다.
코드는 [github](https://github.com/novemberde/git-puller)에 공개되어 있으며 사용 방법은 아래와 같다.

---

```sh
$ npm install -g git-puller
```

### Usage

```sh
Usage: git-puller [options]

Options:

  -V, --version            output the version number
  -d, --directory [value]  Target directory
  -r, --remote [value]     Git Remote (default: origin)
  -b, --branch [value]     Git branch (default: master)
  -h, --help               output usage information

examples:
  git-puller -d ./                       # Current directory
  git-puller -d ../../_my_project        # Other directory
  git-puller -d ./ -r origin -b master   # Specify remote and branch

  gplr -d ./                             # Current directory
  gplr -d ../../_my_project              # Other directory
  gplr -d ./ -r origin -b master         # Specify remote and branch
```

---
### References
---
- [https://github.com/novemberde/git-puller](https://github.com/novemberde/git-puller)

