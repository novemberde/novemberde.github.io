---
title: Direnv를 활용한 프로젝트 별 환경설정하기
tags: [direnv, 환경설정, environment, env, 설정, project]
date: "2019-10-04T11:30:03+00:00"
aliases: ["/general/2019/10/04/Direnv.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

현재 Amazon EKS위에 컨테이너를 배포하기 위한 환경을 구축중이다.
각 운영되는 서비스 또는 Repository마다 환경들이 다르고,
개인으로 운영하는 서비스도 가끔 확인해야하는데 Global로 AWS 또는 K8s 환경 설정을 하면
귀찮아지기 때문에 폴더별로 모든 환경을 관리하고 싶어졌다.

Direnv를 활용하면 directory마다 환경을 따로 관리할 수 있는데, 매번 같은 키를 넣고 관리하기도 귀찮아서
Direnv에서 사용하는 변수들도 따로 관리하고 있다.
다음은 현재 사용하는 패턴이다.

## Direnv 설치하기

다음은 참고한 문서이다.

[direnv로 디렉토리(프로젝트) 별 개발환경 구축하기](https://www.44bits.io/ko/post/direnv_for_managing_directory_environment)


## AWS 설정 옮겨가며 편하게 사용하기

AWS CLI를 계정별로 관리하고 싶을 때 profile 옵션을 사용한다. 하지만 이는 매번 profile을 입력해야하는 번거로움이 있다.
위에 링크를 잘 살펴보면 "전역 설정 파일 .direnvrc"를 찾을 수 있다. 이걸 잘 활용하면 간편하게 디렉터리별 환경 설정을 한 곳에서 관리할 수 있다.

아래는 내가 사용하는 기본적인 .direnvrc 파일이다.

```sh
use_ruby() {
  local ruby_root=$HOME/.rubies/$1
  load_prefix "$ruby_root"
  layout_ruby
}

company_aws() {
  export AWS_ACCESS_KEY_ID=~~~
  export AWS_SECRET_ACCESS_KEY=~~~
}
my_private_aws() {
  export AWS_ACCESS_KEY_ID=~~~
  export AWS_SECRET_ACCESS_KEY=~~~~
  echo "load my_private_aws"
}
```

이렇게 설정해 놓으면 다음부턴 해당 프로젝트 디렉토리에 .envrc 파일에서 다음과 같이 입력하면 해당 환경변수를 불러온다.

```sh
my_private_aws
```

위와같이 저장하고 다시 direnv allow를 해준다.

```sh
$ direnv allow
direnv: loading .envrc
load my_private_aws
direnv: export +AWS_ACCESS_KEY_ID +AWS_SECRET_ACCESS_KEY
```

간단하게 프로젝트 디렉터리 마다 설정을 불러올 수 있다.


## Kubeconfig 따로 관리하기

Kubeconfig는 성격이 달라서 필요한 경우에만 주입해주는게 낫다.
.direnvrc 포함시키는 건 전체적으로 설정되기 때문에 실수할 가능성이 많이 생긴다.
귀찮더라도 스테이징마다 kubeconfig를 설정하는데 관리하는 방식은 다음과 같다.

```sh
$ tree -L 2 ~/.kube
├── cache
│   └── discovery
├── my_private_cluster.yaml
└── company_alpha_cluster.yaml
```

개발이 진행중인 프로젝트 디렉토리에서 다음과 같이 설정한다.

```sh
export KUBECONFIG=~/.kube/my_private_cluster.yaml
```

스테이징마다 별도로 디렉터리 구조를 잡고 있다면 클러스터에 잘못 배포되는 실수를 방지할 수 있다.

## References

- [https://www.44bits.io/ko/post/direnv_for_managing_directory_environment](https://www.44bits.io/ko/post/direnv_for_managing_directory_environment)
