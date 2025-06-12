# GitOps

Kubernetes 클러스터를 운영하는데 필요한 소스 코드입니다.

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