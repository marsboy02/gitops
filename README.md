# GitOps

Kubernetes 클러스터를 운영하는데 필요한 소스 코드입니다.

## 🛠️ 선행조건

이 프로젝트를 실행하기 전에 다음 도구들을 설치해야 합니다.

### 1. Homebrew 설치 (macOS)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. kubectl 설치
```bash
# kubectl 설치
brew install kubectl

# 설치 확인
kubectl version --client
```

### 3. minikube 설치 및 시작
```bash
# minikube 설치
brew install minikube

# minikube 시작
minikube start

# minikube 상태 확인
minikube status

# kubectl이 minikube 클러스터를 가리키는지 확인
kubectl cluster-info
```

### 4. k9s 설치 (선택사항 - 클러스터 모니터링 도구)
```bash
# k9s 설치
brew install k9s

# k9s 실행
k9s
```

### 5. kustomize 설치 (kubectl에 내장되어 있지만 최신 버전 사용 권장)
```bash
# kustomize 최신 버전 설치
brew install kustomize

# 설치 확인
kustomize version
```

### 🔧 추가 유용한 도구들
```bash
# Docker Desktop (minikube에서 docker 드라이버 사용시)
brew install --cask docker

# Helm (패키지 매니저)
brew install helm

# kubectx & kubens (컨텍스트 및 네임스페이스 전환)
brew install kubectx
```

## ⚡ 빠른 시작

선행조건을 모두 설치했다면 다음 명령어로 바로 시작할 수 있습니다:

```bash
# 1. minikube 클러스터 시작
minikube start

# 2. 모든 애플리케이션을 stage 환경에 배포
kubectl apply -k .

# 3. 배포 상태 확인
kubectl get all --all-namespaces -l gitops-project=true

# 4. k9s로 클러스터 모니터링 (선택사항)
k9s
```

## 📁 프로젝트 구조

```
├── projects/
│   ├── nginx/          # Nginx 웹 서버
│   ├── postgres/       # PostgreSQL 데이터베이스
│   └── redis/          # Redis 캐시
├── environments/
│   ├── stage/          # Stage 환경 통합 설정
│   └── prod/           # Production 환경 통합 설정
└── kustomization.yaml  # 기본 설정 (stage 환경 참조)
```

## 🚀 배포 방법

### 모든 프로젝트 한 번에 배포

#### 1. Stage 환경 배포 (기본)
```bash
# 방법 1: 루트에서 기본 설정 사용
kubectl apply -k .

# 방법 2: 직접 stage 환경 지정
kubectl apply -k environments/stage

# 방법 3: kustomize build로 먼저 확인 후 배포
kustomize build . | kubectl apply -f -
```

#### 2. Production 환경 배포
```bash
# 직접 prod 환경 지정
kubectl apply -k environments/prod

# kustomize build로 먼저 확인 후 배포
kustomize build environments/prod | kubectl apply -f -
```

### 개별 프로젝트 배포

필요한 경우 개별 프로젝트만 배포할 수도 있습니다:

```bash
# Nginx만 배포 (stage)
kubectl apply -k projects/nginx/overlay/stage

# PostgreSQL만 배포 (prod)
kubectl apply -k projects/postgres/overlay/prod

# Redis만 배포 (stage)
kubectl apply -k projects/redis/overlay/stage
```

## 📊 배포 상태 확인

```bash
# 모든 네임스페이스의 리소스 확인
kubectl get all --all-namespaces -l gitops-project=true

# 각 프로젝트별 리소스 확인
kubectl get pods -n nginx-sample
kubectl get statefulset -n postgres-system
kubectl get statefulset -n redis-system

# 서비스 확인
kubectl get svc --all-namespaces -l gitops-project=true

# Ingress 확인
kubectl get ingress --all-namespaces
```

## 🧹 리소스 정리

```bash
# 특정 환경의 모든 리소스 삭제
kubectl delete -k environments/stage
kubectl delete -k environments/prod

# 또는 네임스페이스 전체 삭제
kubectl delete namespace nginx-sample postgres-system redis-system
```

## 🔧 Kustomize 장점

- **단일 명령어**: `kubectl apply -k .`로 모든 프로젝트 배포
- **환경별 관리**: stage/prod 환경을 깔끔하게 분리
- **공통 설정**: 모든 리소스에 환경별 라벨/어노테이션 자동 적용
- **Kubernetes 네이티브**: 추가 도구 없이 kubectl과 kustomize만 사용
- **GitOps 친화적**: ArgoCD, Flux 등과 완벽 호환

## 🚨 문제 해결

### minikube 관련 문제

```bash
# minikube 상태 확인
minikube status

# minikube 재시작
minikube stop
minikube start

# minikube 완전 삭제 후 재생성
minikube delete
minikube start

# Docker 드라이버 사용 (권장)
minikube start --driver=docker

# 메모리 부족시
minikube start --memory=4096 --cpus=2
```

### kubectl 컨텍스트 문제

```bash
# 현재 컨텍스트 확인
kubectl config current-context

# minikube 컨텍스트로 전환
kubectl config use-context minikube

# 클러스터 연결 확인
kubectl cluster-info
```

### 서비스 접근 문제

```bash
# minikube 서비스 URL 확인
minikube service list

# 특정 서비스 URL 열기
minikube service <service-name> -n <namespace>

# minikube 터널 (LoadBalancer 타입 서비스용)
minikube tunnel
```

### k9s 사용법

- TUI를 통해서 쿠버네티스 클러스터를 모니터링합니다.

```bash
# k9s 실행
k9s

# 주요 단축키:
# 0-9: 네임스페이스 전환
# :pods, :svc, :deploy: 리소스 타입별 보기
# d: 상세 정보 보기
# l: 로그 보기
# q: 종료
```
