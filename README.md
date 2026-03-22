# Todo Manifest Repository

Todo App의 Kubernetes 매니페스트를 관리하는 GitOps 레포지토리입니다.
ArgoCD가 이 레포를 감시하며, 변경이 감지되면 자동으로 EKS 클러스터에 배포합니다.

## 구조

```
todo-manifest-repository/
├── deployment.yaml    # Deployment (frontend + backend 컨테이너)
├── service.yaml       # Service (ClusterIP, 포트 80)
└── ingress.yaml       # Ingress (ALB, internet-facing)
```

## 초기 세팅 (Fork 후 필수)

### 1. todo-app-repository에서 CI 파이프라인 1회 실행

`todo-app-repository`를 fork한 뒤, GitHub Actions Variables/Secrets를 설정하고 main 브랜치에 push하세요.
CI가 실행되면 Docker 이미지가 ECR에 푸시되고, 이 레포의 `deployment.yaml` 이미지 태그가 자동으로 업데이트됩니다.

### 2. deployment.yaml 이미지 주소 확인

CI가 정상 실행되면 `deployment.yaml`의 이미지가 아래와 같이 변경됩니다:

```yaml
# 변경 전 (플레이스홀더)
image: <ECR_FRONTEND_IMAGE>
image: <ECR_BACKEND_IMAGE>

# 변경 후 (CI가 자동 업데이트)
image: 123456789012.dkr.ecr.us-west-2.amazonaws.com/todo-frontend:abc12345
image: 123456789012.dkr.ecr.us-west-2.amazonaws.com/todo-backend:abc12345
```

> CI 실행 전에 ArgoCD Application을 먼저 생성하면 `<ECR_FRONTEND_IMAGE>` 플레이스홀더 때문에 ImagePullBackOff가 발생합니다.
> 반드시 CI를 먼저 실행하여 실제 이미지 주소로 변경된 후 ArgoCD Application을 생성하세요.

## ArgoCD Application 생성

CI 실행 후 이미지가 업데이트되면, ArgoCD 대시보드에서 Application을 생성합니다:

## 이후 배포 흐름

1. `todo-app-repository`에서 코드 수정 후 main에 push
2. GitHub Actions CI가 새 이미지를 ECR에 푸시
3. CI가 이 레포의 `deployment.yaml` 이미지 태그를 자동 업데이트
4. ArgoCD가 변경 감지 → 자동 Sync → EKS에 배포
