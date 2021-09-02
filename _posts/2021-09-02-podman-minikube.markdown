---
title: "minikube로 docker와 docker-compose를 대체하기"
category: docker
tags: docker, podman, minikube
---

Docker desktop이 유료로 전환된다는 소식이 있다. ([링크](https://www.docker.com/blog/updating-product-subscriptions/))

이 소식을 들은 많은 개발자들은 당황했을거라 생각하는데 언제나 그랬듯이 대안은 있다.
사내에 이미 이런 내용에 대해서 대응하신 분이 있고, 그걸 글로 녹여보려고 한다.(Ian 감사해요)
아래 내용은 Mac에만 해당되며, Window는 별도의 방법을 찾아야할지도 모른다.

---

## Docker Desktop 삭제하기

이건 어렵지 않다. 다음 그림과 같이 docker desktop > Preferences > troubleshootings > uninstall 을 눌러 삭제한다.

![remove-docker-desktop](/images/docker/remove-docker-desktop.png)

그리고 말끔하게 Applications(응용프로그램)에 있는 docker desktop도 삭제한다.

## minikube 설치하기

m1 mac인 경우에는 ["Disable all drivers except "docker" and "ssh" on darwin/arm64" PR](https://github.com/kubernetes/minikube/pull/10452)을 참고하자면
minikube에서는 m1 mac을 사용할 수 없다. docker를 사용하던지 아니면 ssh로 리모트의 VM을 써야한다.
뭐 로컬에 virtualbox나 vmware로 띄우고 ssh를 써도 사용은 되겠지만, localhost에서 사용하기 위한 다른 설정을 하는 건 조금 수고스럽다고 생각된다.

당장은 방법이 없으니 조금만 더 docker desktop을 사용하면 된다.

만약 podman이 apple silicon을 지원하게 된다면 그때 이 문서를 다시 업데이트할 것이다.

### 설치하기

```sh
$ brew install hyperkit
$ brew install minikube
$ minikube start --driver=hyperkit
```

### 확인하기

```sh
$ kubectl get po -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-fb4qn           1/1     Running   0          3m37s
kube-system   etcd-minikube                      1/1     Running   0          3m48s
kube-system   kube-apiserver-minikube            1/1     Running   0          3m48s
kube-system   kube-controller-manager-minikube   1/1     Running   0          3m48s
kube-system   kube-proxy-9k4vr                   1/1     Running   0          3m37s
kube-system   kube-scheduler-minikube            1/1     Running   0          3m48s
kube-system   storage-provisioner                1/1     Running   0          3m50s
```

## Kompose로 docker-compose.yml을 minikube에 올리기

### Kompose 설치하기

```sh
$ brew install kompose
```

### docker-compose.yml을 kompose convert하고 배포하기

**docker.compose.yml 파일**

```yml
version: "2"

services:

  redis-master:
    image: k8s.gcr.io/redis:e2e
    ports:
      - "6379"

  redis-slave:
    image: gcr.io/google_samples/gb-redisslave:v3
    ports:
      - "6379"
    environment:
      - GET_HOSTS_FROM=dns

  frontend:
    image: gcr.io/google-samples/gb-frontend:v4
    ports:
      - "80:80"
    environment:
      - GET_HOSTS_FROM=dns
    labels:
      kompose.service.type: LoadBalancer
```

#### Convert 하기

```sh
$ kompose convert -f docker-compose.yml --out converted.yml
```

#### 배포하기

```sh
$ kubectl apply -f converted.yml

# 확인해보기
$ kubectl get po -A
NAMESPACE     NAME                               READY   STATUS              RESTARTS   AGE
default       frontend-69f6f756d8-4kmpk          1/1     Running             0          47s
default       redis-master-db85b98bc-4kzlm       0/1     ContainerCreating   0          47s
default       redis-slave-69bf546647-8499k       1/1     Running             0          47s
kube-system   coredns-558bd4d5db-fb4qn           1/1     Running             0          25m
kube-system   etcd-minikube                      1/1     Running             0          25m
kube-system   kube-apiserver-minikube            1/1     Running             0          25m
kube-system   kube-controller-manager-minikube   1/1     Running             0          25m
kube-system   kube-proxy-9k4vr                   1/1     Running             0          25m
kube-system   kube-scheduler-minikube            1/1     Running             0          25m
kube-system   storage-provisioner                1/1     Running             0          25m
```

## 후기

docker cli의 명령어들이 익숙한 상황에서 kubectl이 조금 불편하긴 하지만 이미 컨테이너를 사용하는 환경에서는 k8s가 지배하고 있기 때문에
이참에 적응하는 것도 참 좋을 것 같다.
minikube가 워낙 가볍기도 하고 연습하기 좋아서 장기적으로는 이번에 갈아타는 게 좋지 않을까 조심스레 추천해본다.

## References

- [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
- [https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/](https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/)