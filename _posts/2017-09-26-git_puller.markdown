---
title: Git pull all sub repository of certain directory.
category: git
tags: git, pull, subdirectory, directory, sub, renew, 갱신
---
## Summary
---
여러 프로젝트를 동시에 진행하다보니 여러 repository를 한번에 갱신할 필요성이 생겼다. 일일이 하나씩 들어가서 git pull을 하려니 은근 귀찮았다.
그렇기 때문에 특정 디렉터리에 있는 all repositories(sub directories)에 대해서 git을 갱신해주는 코드를 개발하였다.
코드는 [github](https://github.com/novemberde/git-puller)에 공개되어 있으며 사용 방법은 아래와 같다.

I need to pull all repository of certain directory because of multiple project at this moment.


---

```sh
$ npm install -g git-puller
```

### Usage

```sh
$ git-puller <DirectoryName>

# Example
$ git-puller .  # current directory
$ git-puller ../../_my_project # other directory
Execute:  ls -d ../../_my_project/*/
Directory list:
  ../../_my_project/Repository1
  ../../_my_project/Repository2
  ../../_my_project/Repository3
  ../../_my_project/Repository4

Execute:  cd ../../_my_project && git fetch && git pull origin
fatal: Not a git repository (or any of the parent directories): .git

Execute:  cd ../../_my_project/Repository1 && git fetch && git pull origin
Already up-to-date.
...
```

---
### References
---
- [https://github.com/novemberde/git-puller](https://github.com/novemberde/git-puller)

