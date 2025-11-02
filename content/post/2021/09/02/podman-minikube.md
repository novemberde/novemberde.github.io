---
title: "minikube로 docker와 docker-compose를 대체하기"
tags: [docker, podman, minikube, kubernetes, docker-desktop, colima, rancher-desktop]
date: "2021-09-12T11:30:03+00:00"
aliases: ["/docker/2021/09/02/podman-minikube.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

2021년 8월, Docker Desktop이 대기업(직원 250명 이상 또는 연매출 1천만 달러 이상)에 대해 유료화된다는 발표가 나왔다. 많은 개발자들이 당황했지만, 이는 오히려 로컬 개발 환경을 재평가하고 쿠버네티스 기반 워크플로우로 전환할 좋은 기회가 되었다.

이 글에서는 단순히 Docker Desktop을 대체하는 것을 넘어, 로컬 개발 환경을 프로덕션과 더 가깝게 만들고 쿠버네티스 생태계에 자연스럽게 적응하는 방법을 다룬다.

## Docker Desktop이 필요했던 이유

먼저 Docker Desktop이 무엇을 제공했는지 이해해야 한다:

1. **Docker Engine**: 컨테이너를 실행하는 핵심 엔진
2. **VM 통합**: macOS/Windows에서 Linux 컨테이너를 실행하기 위한 경량 VM
3. **Docker Compose**: 멀티 컨테이너 애플리케이션 정의 및 실행
4. **GUI**: 컨테이너 관리를 위한 시각적 인터페이스
5. **Kubernetes**: 로컬 쿠버네티스 클러스터 (옵션)
6. **편의 기능**: 파일 공유, 네트워크 자동 설정, 자동 업데이트

이 중 많은 부분이 오픈소스 도구들로 대체 가능하다. 문제는 "어떻게"가 아니라 "왜"와 "무엇으로"다.

## 왜 minikube인가?

Docker Desktop 대안으로 여러 선택지가 있다:

- **Podman**: Docker 호환 컨테이너 런타임
- **Colima**: Lima 기반의 경량 Docker 대체제
- **Rancher Desktop**: 완전한 GUI를 제공하는 쿠버네티스 환경
- **minikube**: 로컬 쿠버네티스 클러스터

minikube를 선택하는 이유:

1. **프로덕션 패리티**: 대부분의 프로덕션 환경이 쿠버네티스다. 로컬에서도 쿠버네티스를 쓰면 환경 차이로 인한 문제가 줄어든다.
2. **학습 곡선**: Docker Compose만 알던 개발자가 쿠버네티스를 배울 수 있는 안전한 환경이다.
3. **확장성**: 단순한 컨테이너 실행부터 복잡한 마이크로서비스까지 대응 가능하다.
4. **에코시스템**: Helm, Kustomize 등 쿠버네티스 도구를 로컬에서 테스트할 수 있다.

단점도 명확하다:
- Docker CLI보다 kubectl이 복잡하다
- 리소스를 더 많이 사용한다
- 초기 학습이 필요하다

## 설치 및 초기 설정

### macOS (Intel)

```bash
# Hyperkit 드라이버 설치 (가벼운 VM)
$ brew install hyperkit

# minikube 설치
$ brew install minikube

# kubectl 설치 (이미 있다면 생략)
$ brew install kubectl

# minikube 시작
$ minikube start --driver=hyperkit --memory=4096 --cpus=2

# Docker 환경 설정 (선택사항 - Docker CLI를 minikube와 함께 사용)
$ eval $(minikube docker-env)
```

### macOS (Apple Silicon / M1/M2/M3)

Apple Silicon은 몇 가지 제약이 있다:

```bash
# Docker 드라이버 사용 (권장)
$ brew install minikube

# Docker Desktop이 없다면 Colima 설치
$ brew install colima
$ colima start

# minikube 시작 (Docker 드라이버)
$ minikube start --driver=docker --memory=4096 --cpus=2

# 또는 QEMU 드라이버 (더 나은 격리)
$ brew install qemu
$ minikube start --driver=qemu --memory=4096 --cpus=2
```

**중요**: 2021년 당시 M1 지원이 제한적이었지만, 2024년 현재는 Docker와 QEMU 드라이버로 안정적으로 동작한다.

### Linux

```bash
# minikube 설치
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Docker 드라이버 사용
$ minikube start --driver=docker

# 또는 KVM2 (더 나은 성능)
$ sudo apt-get install libvirt-daemon-system libvirt-clients qemu-kvm
$ minikube start --driver=kvm2
```

### 설정 확인

```bash
# 클러스터 상태 확인
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

# 노드 확인
$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.28.3

# Pod 확인
$ kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-5dd5756b68-xxxxx          1/1     Running   0          2m
kube-system   etcd-minikube                      1/1     Running   0          2m
kube-system   kube-apiserver-minikube            1/1     Running   0          2m
kube-system   kube-controller-manager-minikube   1/1     Running   0          2m
kube-system   kube-proxy-xxxxx                   1/1     Running   0          2m
kube-system   kube-scheduler-minikube            1/1     Running   0          2m
kube-system   storage-provisioner                1/1     Running   0          2m
```

## Docker Compose에서 Kubernetes로 마이그레이션

### 방법 1: Kompose - 자동 변환

Kompose는 docker-compose.yml을 쿠버네티스 매니페스트로 변환한다.

```bash
# Kompose 설치
$ brew install kompose  # macOS
$ curl -L https://github.com/kubernetes/kompose/releases/download/v1.31.2/kompose-linux-amd64 -o kompose  # Linux
$ chmod +x kompose && sudo mv kompose /usr/local/bin/
```

**예제: 간단한 웹 애플리케이션**

```yaml
# docker-compose.yml
version: "3"

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    environment:
      - NGINX_HOST=localhost
      - NGINX_PORT=80

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

  api:
    image: node:18-alpine
    working_dir: /app
    volumes:
      - ./api:/app
    command: npm start
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - REDIS_HOST=redis
    depends_on:
      - redis

volumes:
  redis-data:
```

**Kompose로 변환**

```bash
# 변환 (개별 파일로)
$ kompose convert

# 변환 (하나의 파일로)
$ kompose convert --out k8s-manifests.yaml

# 변환 결과 확인
$ ls -la
-rw-r--r--  api-deployment.yaml
-rw-r--r--  api-service.yaml
-rw-r--r--  redis-deployment.yaml
-rw-r--r--  redis-service.yaml
-rw-r--r--  redis-data-persistentvolumeclaim.yaml
-rw-r--r--  web-deployment.yaml
-rw-r--r--  web-service.yaml
```

**배포**

```bash
# 전체 배포
$ kubectl apply -f .

# 또는 특정 파일만
$ kubectl apply -f web-deployment.yaml
$ kubectl apply -f web-service.yaml

# 상태 확인
$ kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/api-xxxxx              1/1     Running   0          30s
pod/redis-xxxxx            1/1     Running   0          30s
pod/web-xxxxx              1/1     Running   0          30s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/api          ClusterIP   10.96.1.1       <none>        3000/TCP   30s
service/redis        ClusterIP   10.96.1.2       <none>        6379/TCP   30s
service/web          ClusterIP   10.96.1.3       <none>        80/TCP     30s
```

### 방법 2: 수동 변환 - 더 나은 제어

Kompose가 생성한 매니페스트는 기본적인 수준이다. 프로덕션에 가까운 설정을 위해서는 수동 조정이 필요하다.

**개선된 Deployment 예제**

```yaml
# web-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: web
spec:
  replicas: 2  # 고가용성
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:  # 리소스 제한
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        livenessProbe:  # 헬스체크
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 5
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        hostPath:
          path: /Users/username/project/html  # minikube에서 호스트 경로 마운트
          type: Directory
```

**Service 노출 방법**

```yaml
# web-service.yaml - LoadBalancer 타입 (minikube tunnel 필요)
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```bash
# LoadBalancer를 사용하려면 minikube tunnel 실행 (별도 터미널)
$ minikube tunnel

# 이제 localhost:80으로 접근 가능
$ curl http://localhost
```

**또는 NodePort 사용**

```yaml
# web-service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080  # 30000-32767 범위
```

```bash
# NodePort로 접근
$ minikube service web --url
http://192.168.64.2:30080

# 브라우저에서 열기
$ minikube service web
```

**Ingress 사용 (권장)**

```bash
# Ingress 애드온 활성화
$ minikube addons enable ingress

# Ingress 리소스 생성
$ cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: web.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
EOF

# /etc/hosts 수정
$ echo "$(minikube ip) web.local" | sudo tee -a /etc/hosts

# 접근
$ curl http://web.local
```

## 실전 시나리오

### 시나리오 1: 마이크로서비스 개발 환경

여러 서비스가 상호작용하는 환경:

```bash
# Namespace로 프로젝트 분리
$ kubectl create namespace myapp

# Context 설정
$ kubectl config set-context --current --namespace=myapp

# ConfigMap으로 설정 관리
$ kubectl create configmap app-config \
  --from-literal=DB_HOST=postgres \
  --from-literal=REDIS_HOST=redis \
  --from-literal=API_URL=http://api:3000

# Secret으로 민감 정보 관리
$ kubectl create secret generic app-secrets \
  --from-literal=DB_PASSWORD=secret123 \
  --from-literal=API_KEY=abc123

# 전체 스택 배포
$ kubectl apply -f manifests/
```

**개발 워크플로우**

```bash
# 1. 코드 수정
$ vim api/index.js

# 2. 이미지 빌드 (minikube Docker 환경 사용)
$ eval $(minikube docker-env)
$ docker build -t myapp/api:dev ./api

# 3. Pod 재시작
$ kubectl rollout restart deployment/api

# 4. 로그 확인
$ kubectl logs -f deployment/api

# 5. 디버깅이 필요하면 Pod 내부 접근
$ kubectl exec -it deployment/api -- sh
```

### 시나리오 2: Helm으로 복잡한 애플리케이션 관리

```bash
# Helm 설치
$ brew install helm

# 예제: PostgreSQL 설치
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install my-postgres bitnami/postgresql \
  --set auth.username=myuser \
  --set auth.password=mypassword \
  --set auth.database=mydb

# 연결 정보 확인
$ export POSTGRES_PASSWORD=$(kubectl get secret --namespace default my-postgres-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
$ kubectl run my-postgres-postgresql-client --rm --tty -i --restart='Never' --namespace default \
  --image docker.io/bitnami/postgresql:16 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host my-postgres-postgresql -U postgres -d mydb -p 5432
```

### 시나리오 3: 로컬 개발과 CI/CD 파이프라인 통합

```yaml
# .github/workflows/test.yml
name: Test

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Start minikube
        uses: medyagh/setup-minikube@latest

      - name: Deploy application
        run: |
          kubectl apply -f k8s/
          kubectl wait --for=condition=ready pod -l app=myapp --timeout=300s

      - name: Run tests
        run: |
          kubectl exec deployment/myapp -- npm test
```

## Docker Compose 대비 Kubernetes의 차이점

### 개념 매핑

| Docker Compose | Kubernetes | 설명 |
|---------------|------------|------|
| `services` | `Deployment` + `Service` | 컨테이너 실행 + 네트워크 노출 |
| `volumes` | `PersistentVolumeClaim` | 영구 스토리지 |
| `networks` | `Service` / `NetworkPolicy` | 네트워크 분리 |
| `depends_on` | `initContainers` | 시작 순서 제어 |
| `environment` | `ConfigMap` / `Secret` | 환경변수 관리 |
| `.env` 파일 | `ConfigMap` | 설정 파일 |

### 주요 차이점

**1. 선언적 vs 명령적**

```bash
# Docker Compose (명령적 스타일)
$ docker-compose up -d
$ docker-compose scale web=3
$ docker-compose down

# Kubernetes (선언적 스타일)
$ kubectl apply -f deployment.yaml  # 원하는 상태를 선언
$ kubectl scale deployment web --replicas=3
$ kubectl delete -f deployment.yaml
```

**2. 헬스체크와 재시작 정책**

```yaml
# Docker Compose
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: always

# Kubernetes (더 강력한 옵션)
spec:
  containers:
  - name: web
    livenessProbe:  # 프로세스가 살아있는지
      httpGet:
        path: /health
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 10
    readinessProbe:  # 트래픽을 받을 준비가 되었는지
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    startupProbe:  # 애플리케이션이 시작되었는지
      httpGet:
        path: /startup
        port: 80
      failureThreshold: 30
      periodSeconds: 10
  restartPolicy: Always
```

**3. 리소스 관리**

```yaml
# Kubernetes는 리소스 제한을 명시적으로 관리
spec:
  containers:
  - name: web
    resources:
      requests:  # 최소 보장
        memory: "256Mi"
        cpu: "500m"
      limits:  # 최대 제한
        memory: "512Mi"
        cpu: "1000m"
```

## 유용한 minikube 명령어

```bash
# 대시보드 열기 (GUI)
$ minikube dashboard

# 애드온 목록 및 활성화
$ minikube addons list
$ minikube addons enable metrics-server  # 리소스 모니터링
$ minikube addons enable ingress         # Ingress 컨트롤러
$ minikube addons enable registry        # 로컬 레지스트리

# 로컬 레지스트리 사용
$ eval $(minikube docker-env)
$ docker build -t myapp:latest .
# 이제 ImagePullPolicy: Never로 로컬 이미지 사용 가능

# SSH 접속
$ minikube ssh

# IP 확인
$ minikube ip

# Service URL 확인
$ minikube service <service-name> --url

# 로그 확인
$ minikube logs

# 클러스터 중지/시작/삭제
$ minikube stop
$ minikube start
$ minikube delete

# 프로파일 관리 (여러 클러스터)
$ minikube start -p dev-cluster
$ minikube start -p test-cluster
$ minikube profile list
$ minikube profile dev-cluster
```

## 성능 최적화

### 1. 적절한 리소스 할당

```bash
# CPU와 메모리 조정
$ minikube start --cpus=4 --memory=8192

# 디스크 크기 조정
$ minikube start --disk-size=50g
```

### 2. Docker 환경 재사용

```bash
# minikube의 Docker 데몬 사용 (이미지 빌드 속도 향상)
$ eval $(minikube docker-env)

# 원상복구
$ eval $(minikube docker-env -u)
```

### 3. 캐시 활용

```bash
# 이미지 사전 로드
$ minikube cache add redis:alpine
$ minikube cache add nginx:alpine

# 캐시 목록
$ minikube cache list
```

### 4. 멀티 노드 클러스터 (테스트용)

```bash
# 3노드 클러스터 생성
$ minikube start --nodes=3

# 노드 확인
$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
minikube       Ready    control-plane   2m    v1.28.3
minikube-m02   Ready    <none>          1m    v1.28.3
minikube-m03   Ready    <none>          1m    v1.28.3
```

## 트러블슈팅

### 1. minikube가 시작되지 않음

```bash
# 로그 확인
$ minikube logs

# 완전히 삭제하고 재시작
$ minikube delete --all --purge
$ rm -rf ~/.minikube
$ minikube start

# 특정 쿠버네티스 버전 사용
$ minikube start --kubernetes-version=v1.27.0
```

### 2. 이미지를 찾을 수 없음

```bash
# 로컬 이미지 사용 시 ImagePullPolicy 설정
spec:
  containers:
  - name: myapp
    image: myapp:latest
    imagePullPolicy: Never  # 또는 IfNotPresent
```

### 3. PersistentVolume 문제

```bash
# minikube는 hostPath를 기본 스토리지 클래스로 사용
$ kubectl get storageclass
NAME                 PROVISIONER                RECLAIMPOLICY
standard (default)   k8s.io/minikube-hostpath   Delete

# PVC 상태 확인
$ kubectl get pvc
$ kubectl describe pvc <pvc-name>
```

### 4. Service에 접근할 수 없음

```bash
# LoadBalancer 타입이면 tunnel 필요
$ minikube tunnel  # 별도 터미널에서 실행

# NodePort 타입이면 URL 확인
$ minikube service <service-name> --url

# Ingress 사용 시 IP 확인
$ kubectl get ingress
$ minikube ip
```

### 5. 리소스 부족

```bash
# 현재 리소스 사용량 확인
$ kubectl top nodes
$ kubectl top pods

# 사용하지 않는 리소스 정리
$ kubectl delete pod --field-selector=status.phase==Succeeded
$ kubectl delete pod --field-selector=status.phase==Failed

# Docker 이미지 정리
$ eval $(minikube docker-env)
$ docker system prune -a
```

## 대안 비교

### Colima

```bash
# 설치 및 시작
$ brew install colima
$ colima start

# 장점: Docker Desktop과 거의 동일한 경험
# 단점: Kubernetes 지원이 제한적
```

### Rancher Desktop

- GUI 기반 (Docker Desktop과 유사)
- Kubernetes 기본 제공
- containerd 또는 dockerd 선택 가능
- 윈도우, macOS, Linux 지원

### Podman

```bash
# macOS에서 Podman
$ brew install podman
$ podman machine init
$ podman machine start

# Docker 호환 명령어
$ podman run -d nginx
$ podman-compose up

# 장점: rootless 실행, 보안성
# 단점: Docker 완벽 호환 아님
```

### Kind (Kubernetes in Docker)

```bash
# 설치
$ brew install kind

# 클러스터 생성
$ kind create cluster

# 장점: 매우 가볍고 빠름, CI/CD에 최적
# 단점: 기능이 제한적
```

## 언제 무엇을 사용할까?

| 상황 | 추천 도구 | 이유 |
|-----|---------|------|
| 단순 컨테이너 실행 | Colima, Podman | 오버헤드가 적고 Docker와 동일한 경험 |
| 로컬 쿠버네티스 학습 | minikube | 풍부한 기능과 문서 |
| 프로덕션 유사 환경 | minikube, Rancher Desktop | 실제 쿠버네티스와 가장 유사 |
| CI/CD 파이프라인 | Kind, minikube | 빠른 시작과 종료 |
| 멀티 클러스터 테스트 | minikube (프로파일) | 여러 클러스터 동시 관리 |
| GUI 선호 | Rancher Desktop | Docker Desktop과 유사한 UX |

## 마이그레이션 체크리스트

로컬 개발 환경을 minikube로 전환할 때:

- [ ] Docker Compose 파일을 Kubernetes 매니페스트로 변환
- [ ] 볼륨 마운트를 PersistentVolumeClaim으로 변경
- [ ] 환경변수를 ConfigMap/Secret으로 이동
- [ ] 헬스체크 추가 (liveness, readiness probes)
- [ ] 리소스 제한 설정 (requests, limits)
- [ ] 서비스 노출 방법 결정 (LoadBalancer, NodePort, Ingress)
- [ ] 로그 수집 방법 정의 (kubectl logs, 로그 집계)
- [ ] 개발 워크플로우 문서화 (빌드, 배포, 디버깅)
- [ ] 팀원들에게 kubectl 기본 사용법 교육
- [ ] CI/CD 파이프라인 업데이트

## 결론

Docker Desktop의 유료화는 불편한 변화처럼 보였지만, 실제로는 더 나은 개발 환경으로 가는 계기가 되었다. minikube를 사용하면:

1. **프로덕션과 동일한 환경**: 로컬에서 쿠버네티스를 사용하면 "내 컴퓨터에서는 되는데" 문제가 줄어든다.
2. **쿠버네티스 학습**: 자연스럽게 쿠버네티스 개념과 도구를 익힐 수 있다.
3. **더 나은 리소스 관리**: 명시적인 리소스 제한으로 프로덕션 문제를 사전에 발견한다.
4. **확장 가능한 워크플로우**: Helm, Kustomize 등 엔터프라이즈 도구를 로컬에서 사용한다.

초기 학습 곡선이 있지만, 장기적으로는 더 견고하고 확장 가능한 개발 환경을 구축할 수 있다. Docker Compose에 익숙하다면 Kompose로 시작해서 점진적으로 쿠버네티스 기능을 추가하는 것을 추천한다.

## References

- [minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Kompose - Kubernetes + Compose](https://kompose.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Desktop Alternatives](https://collabnix.com/top-10-docker-desktop-alternatives/)
- [Helm Package Manager](https://helm.sh/)
- [Colima - Container runtimes on macOS](https://github.com/abiosoft/colima)
- [Rancher Desktop](https://rancherdesktop.io/)
