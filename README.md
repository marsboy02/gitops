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

### 6. Helm 설치 (Addons 배포에 필요)
```bash
# Helm 설치
brew install helm

# 설치 확인
helm version
```

### 🔧 추가 유용한 도구들
```bash
# Docker Desktop (minikube에서 docker 드라이버 사용시)
brew install --cask docker

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
├── addons/           # GitOps 도구 및 부가 서비스
│   ├── argo-cd/      # Argo CD (GitOps 배포 도구)
│   ├── harbor/       # Harbor (컨테이너 레지스트리)
│   └── redash/       # Redash (BI 데이터 시각화 도구)
├── projects/
│   ├── nginx/        # Nginx 웹 서버
│   ├── postgres/     # PostgreSQL 데이터베이스
│   └── redis/        # Redis 캐시
├── environments/
│   ├── stage/        # Stage 환경 통합 설정
│   └── prod/         # Production 환경 통합 설정
└── kustomization.yaml  # 기본 설정 (stage 환경 참조)
```

## 🚀 배포 방법

### 인프라 애드온 배포

애드온은 GitOps 환경을 구축하고 운영하는데 필요한 도구들입니다. Helm을 사용하여 배포합니다.

#### 1. Argo CD 배포 (GitOps 배포 도구)

```bash
# 의존성 업데이트
helm dependency update ./addons/argo-cd

# Argo CD 배포
helm install argocd ./addons/argo-cd -n argocd --create-namespace

# 배포 상태 확인
kubectl get pods -n argocd

# Argo CD UI 접근 (포트 포워딩)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# 초기 admin 비밀번호 확인
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

UI는 https://localhost:8080 에서 접근 가능합니다.

#### 2. Harbor 배포 (컨테이너 레지스트리)

```bash
# Harbor 배포
helm install harbor ./addons/harbor -n harbor --create-namespace

# 배포 상태 확인
kubectl get pods -n harbor

# Harbor UI 접근 (포트 포워딩)
kubectl port-forward svc/harbor-portal -n harbor 8081:80
```

Harbor의 기본 접근 정보:
- 사용자명: admin
- 비밀번호: Harbor12345 (커스텀 values.yaml에서 변경 가능)

#### 3. Redash 배포 (BI 데이터 시각화 도구)

```bash
# Redash 배포
helm install redash ./addons/redash -n redash --create-namespace

# 배포 상태 확인
kubectl get pods -n redash

# Redash UI 접근 (포트 포워딩)
kubectl port-forward svc/redash-web -n redash 8082:80
```

Redash의 기본 접근 정보:
- 첫 접속 시 관리자 계정을 설정하는 화면이 표시됩니다.

#### GitOps 방식으로 다른 애드온 관리

Argo CD 설치 후, 다음과 같이 GitOps 방식으로 다른 애드온을 관리할 수 있습니다:

1. Argo CD UI에서 애플리케이션 생성
2. Git 레포지토리로 현재 레포지토리 URL 지정
3. 경로(Path)로 `addons/harbor` 또는 `addons/redash` 지정
4. 배포 네임스페이스 지정 후 생성

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

# 애드온 확인
kubectl get pods -n argocd
kubectl get pods -n harbor
kubectl get pods -n redash
```

## 🧹 리소스 정리

```bash
# 특정 환경의 모든 리소스 삭제
kubectl delete -k environments/stage
kubectl delete -k environments/prod

# 애드온 삭제
helm uninstall argocd -n argocd
helm uninstall harbor -n harbor
helm uninstall redash -n redash

# 또는 네임스페이스 전체 삭제
kubectl delete namespace argocd harbor redash nginx-sample postgres-system redis-system
```

## 🔧 Kustomize와 Helm의 장점

### Kustomize (프로젝트 배포)
- **단일 명령어**: `kubectl apply -k .`로 모든 프로젝트 배포
- **환경별 관리**: stage/prod 환경을 깔끔하게 분리
- **공통 설정**: 모든 리소스에 환경별 라벨/어노테이션 자동 적용
- **Kubernetes 네이티브**: 추가 도구 없이 kubectl과 kustomize만 사용
- **GitOps 친화적**: ArgoCD, Flux 등과 완벽 호환

### Helm (애드온 배포)
- **복잡한 앱 배포**: 단일 명령어로 복잡한 애플리케이션 배포
- **구성 가능성**: values.yaml을 통한 세부 구성 조정
- **템플릿 엔진**: 강력한 템플릿 기능으로 동적 매니페스트 생성
- **릴리스 관리**: 설치, 업그레이드, 롤백 기능
- **패키지 관리**: 의존성 관리 기능 제공

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
