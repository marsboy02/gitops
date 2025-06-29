# Kubernetes Addons

이 디렉토리는 Kubernetes 클러스터에 배포할 수 있는 핵심 애드온 애플리케이션들의 Helm 차트를 포함하고 있습니다. 이 애드온들은 GitOps 관리를 위한 기반 인프라를 제공합니다.

## 포함된 애드온

### Argo CD

[Argo CD](https://argo-cd.readthedocs.io/)는 Kubernetes를 위한 선언적 GitOps 지속적 배포 도구입니다. 이 레포지토리에 포함된 버전은 v3.0.5 입니다.

### Harbor

[Harbor](https://goharbor.io/)는 컨테이너 이미지 및 Helm 차트를 저장, 서명, 스캔할 수 있는 오픈소스 컨테이너 레지스트리 플랫폼입니다.

### Redash

[Redash](https://redash.io/)는 데이터를 쉽게 시각화하고 공유할 수 있는 오픈소스 BI(Business Intelligence) 도구입니다. 다양한 데이터 소스에 연결하여 SQL 쿼리를 실행하고 결과를 대시보드로 시각화할 수 있습니다.

## 사전 요구사항

- Kubernetes 클러스터 (v1.25 이상)
- [Helm](https://helm.sh/) v3.0.0+
- kubectl이 클러스터에 접근 가능하도록 설정

## 애드온 배포 방법

### 직접 Helm을 사용하는 방법

#### Argo CD 배포

```bash
# 1. Helm 의존성 업데이트
helm dependency update ./addons/argo-cd

# 2. Argo CD 배포
helm install argocd ./addons/argo-cd -n argocd --create-namespace

# 3. 배포 상태 확인
kubectl get pods -n argocd

# 4. Argo CD UI 접근 (포트 포워딩 사용)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# 5. 초기 admin 비밀번호 얻기
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

이제 https://localhost:8080 에서 Argo CD UI에 접근할 수 있습니다.

#### Harbor 배포

```bash
# 1. Harbor 배포
helm install harbor ./addons/harbor -n harbor --create-namespace

# 2. 배포 상태 확인
kubectl get pods -n harbor

# 3. Harbor UI 접근 (포트 포워딩 사용, 설정에 따라 포트 번호 변경 필요)
kubectl port-forward svc/harbor-portal -n harbor 8081:80
```

Harbor의 기본 접근 정보:
- 사용자명: admin
- 비밀번호: Harbor12345 (커스텀 values.yaml에서 변경 가능)

#### Redash 배포

```bash
# 1. Redash 배포
helm install redash ./addons/redash -n redash --create-namespace

# 2. 배포 상태 확인
kubectl get pods -n redash

# 3. Redash UI 접근 (포트 포워딩 사용)
kubectl port-forward svc/redash-web -n redash 8082:80
```

Redash의 기본 접근 정보:
- 첫 접속 시 관리자 계정을 설정하는 화면이 표시됩니다.

### GitOps 방식으로 배포 (권장)

GitOps 워크플로우에서는:

1. 먼저 Argo CD를 설치합니다:
```bash
helm install argocd ./addons/argo-cd -n argocd --create-namespace
```

2. Argo CD에 접속한 후, 이 Git 레포지토리를 소스로 등록합니다.

3. Argo CD Applications을 생성하여 나머지 애드온들(Harbor, Redash 등)을 자동으로 배포하도록 설정합니다.

## 커스터마이징

각 애드온의 배포 설정을 커스터마이징하려면:

```bash
# 커스텀 values 파일 사용하기
helm install argocd ./addons/argo-cd -n argocd --create-namespace -f my-argocd-values.yaml
helm install redash ./addons/redash -n redash --create-namespace -f my-redash-values.yaml
```

## 참고 문서

- [Argo CD 공식 문서](https://argo-cd.readthedocs.io/)
- [Harbor 공식 문서](https://goharbor.io/docs/)
- [Redash 공식 문서](https://redash.io/help/)
- [Helm 공식 문서](https://helm.sh/docs/)