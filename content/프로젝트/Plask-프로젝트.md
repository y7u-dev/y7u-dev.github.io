# passorder-infra 프로젝트

## 프로젝트 개요

패스오더 카페 주문 플랫폼을 모방한 MSA 인프라 구성 프로젝트.
페이타랩 DevOps Engineer 포트폴리오 목적으로 실제 기술 스택을 직접 구현했습니다.

- **기간**: 2026.04.13 ~ 2026.04.18
- **GitHub**: https://github.com/y7u-dev/passorder-infra
- **목표**: 페이타랩 기술 스택(AWS + EKS + Jenkins + Argo CD + Prometheus + Datadog) 직접 구현

---

## 기술 스택

| 분류 | 기술 |
| --- | --- |
| 언어 | Node.js (Express) |
| 컨테이너 | Docker |
| 클러스터 | AWS EKS (t3.small, On-Demand) |
| 이미지 저장소 | AWS ECR |
| IaC | Terraform |
| CI | Jenkins (EC2) |
| CD | Argo CD (GitOps) |
| 패키지 관리 | Helm |
| 메트릭 모니터링 | Prometheus + Grafana |
| APM | Datadog |
| 오토스케일링 | Kubernetes HPA |

---

## 아키텍처 설계 과정

### 전체 흐름

```
개발자(로컬)
    │ git push
    ▼
GitHub (passorder-infra)
    │ Webhook
    ▼
Jenkins (EC2 t3.small)        ← CI
    │ docker build + push
    ▼
ECR (이미지 저장소)
    │ values.yaml 태그 업데이트
    ▼
Argo CD (EKS 내부)            ← CD (GitOps)
    │ Helm Chart sync
    ▼
EKS Cluster (On-Demand)       ← 실행 환경
    ├── passorder-api Pod
    ├── Prometheus + Grafana   ← 메트릭 모니터링
    └── Datadog Agent          ← APM + 인프라 모니터링
```

### 주요 설계 결정

**Jenkins를 관리형 CI 대신 선택한 이유**

배포 빈도가 높을수록 GitHub Actions 같은 관리형 CI는 비용이 선형으로 증가하지만, Jenkins는 EC2 고정 비용만 발생합니다. 페이타랩이 Jenkins를 선택한 것도 같은 맥락으로 판단했습니다.

**Spot Instance + On-Demand 혼합 전략**

순수 Spot Instance는 AWS가 회수 시 서비스 중단 리스크가 있습니다. 실제 운영에서는 On-Demand 최소 1개 + Spot 혼합 구성을 권장합니다. (신규 계정 Spot quota 제한으로 현재 On-Demand로 구성)

**GitOps 패턴 (Jenkins + Argo CD 역할 분리)**

- Jenkins: 코드 → Docker 이미지 빌드 + ECR push (CI)
- Argo CD: Git 상태를 클러스터에 반영 (CD)
- 배포 이력이 Git 커밋으로 남아 롤백이 용이합니다.

---

## CI/CD 흐름

```
1. git push → GitHub Webhook → Jenkins 자동 트리거
2. Jenkins: docker build → ECR push → values.yaml 이미지 태그 업데이트
3. Argo CD: values.yaml 변경 감지 → Helm Chart 렌더링 → EKS 배포
```

---

## 보안 설계

| 레이어 | 적용 내용 |
| --- | --- |
| 컨테이너 | node 유저 실행 (root 권한 제거), alpine 경량 이미지 |
| 네트워크 | EKS 노드 프라이빗 서브넷 배치, Security Group 최소 포트 개방 |
| IAM | 클러스터/노드별 최소 권한 Role 분리 |
| 이미지 | ECR scan_on_push 활성화 |
| 수명주기 | ECR lifecycle policy로 최근 10개 이미지만 유지 |

---

## 트러블슈팅 상세 기록

### 1. EKS 노드 그룹 생성 실패 (Spot Instance quota)

**문제**: EKS 노드 그룹 생성 시 `AsgInstanceLaunchFailures` 에러 발생

**원인 분석**: AWS 신규 계정은 Spot Instance vCPU quota가 기본 0으로 설정됨. CloudTrail 로그와 EKS 콘솔 이벤트에서 확인.

**해결**: On-Demand로 capacity_type 변경

**개선 / 배운 점**: Spot은 비용 절감 효과가 크지만 신규 계정 quota 제한과 회수 리스크가 있음. 실제 운영에서는 On-Demand 최소 1개 + Spot 혼합 전략이 적합함을 인지

---

### 2. t3.medium Free Tier 미지원

**문제**: t3.medium으로 노드 그룹 생성 시 `InvalidParameterCombination` 에러

**원인 분석**: AWS 신규 계정의 Free Tier는 특정 인스턴스 타입만 허용. 아래 명령어로 직접 확인:

```bash
aws ec2 describe-instance-types \
  --filters "Name=free-tier-eligible,Values=true" \
  --query "InstanceTypes[*].InstanceType" \
  --output table
```

**해결**: t3.small로 변경

**개선 / 배운 점**: 인프라 설계 시 계정 제약 조건을 사전에 확인하는 것이 중요. AWS CLI로 직접 조회하는 방법 습득

---

### 3. Too many pods

**문제**: Datadog Agent Pod가 Pending 상태 지속

**원인 분석**: `kubectl describe pod`의 Events 섹션에서 `0/2 nodes are available: Too many pods` 확인. t3.small은 ENI 기반으로 노드당 최대 Pod 수가 11개로 제한됨.

**해결**: 노드 수를 2개 → 3개로 증설

**개선 / 배운 점**: 인스턴스 타입 선택 시 Pod 밀도(ENI 한도)를 사전에 고려해야 함. 장기적으로 HPA + Cluster Autoscaler 도입으로 노드 수를 동적으로 관리하는 구조가 필요

---

### 4. .terraform/ Git 추적 문제

**문제**: `git push` 시 685MB 파일로 인해 GitHub push 거부

**원인 분석**: `.gitignore` 생성 전에 `git add .`를 실행해서 프로바이더 바이너리가 Git에 추적됨. Git은 이미 추적 중인 파일을 `.gitignore`로 제외하지 못함.

**해결**:

```bash
git rm -r --cached terraform/.terraform
git reset --soft <이전 커밋>
git add terraform/provider.tf terraform/variables.tf  # 필요한 파일만 선택
git commit -m "feat: add Terraform configuration"
git push origin main
```

**개선 / 배운 점**: 새 폴더/도구 추가 전 `.gitignore`를 먼저 작성하는 습관이 중요

---

### 5. Jenkins git push 실패 (detached HEAD)

**문제**: Jenkins 파이프라인에서 `git push origin main` 실패. `error: src refspec main does not match any`

**원인 분석**: Jenkins `checkout scm`은 브랜치가 아닌 특정 커밋 SHA를 직접 체크아웃하기 때문에 HEAD가 브랜치를 가리키지 않음 (detached HEAD 상태). `git branch` 명령으로 확인.

**해결**: Jenkinsfile의 git push 전에 아래 추가:

```bash
git checkout -B main origin/main
```

**개선 / 배운 점**: Jenkins의 Git checkout 동작 방식을 이해하게 됨. `checkout scm` vs 직접 checkout의 차이를 인지하고, CI 환경에서는 항상 명시적으로 브랜치를 설정해야 함

---

### 6. Docker npm permission denied

**문제**: Jenkins 빌드 시 `npm ci` 실행에서 `EACCES: permission denied, mkdir '/app/node_modules'` 에러

**원인 분석**: `USER node` 선언 이후 명령어가 node 유저 권한으로 실행되는데, `WORKDIR /app`이 root 소유라 node 유저가 파일을 쓸 수 없음.

**해결**: `USER node`를 Dockerfile 마지막으로 이동

```docker
# 변경 전
USER node
WORKDIR /app
RUN npm ci  # node 유저가 /app에 쓸 수 없음

# 변경 후
WORKDIR /app
RUN npm ci  # root로 실행
USER node   # 실행만 node 유저로
```

**개선 / 배운 점**: Linux 권한 구조와 Docker 레이어 실행 순서를 이해하게 됨. 보안을 위해 컨테이너 실행 유저를 낮추되, 빌드 단계에서의 권한 분리를 명확히 설계해야 함

---

### 7. Helm values.yaml 변경 미반영

**문제**: `values.yaml`의 이미지 태그를 변경해도 Pod가 새로 뜨지 않음

**원인 분석**: `helm template` 명령으로 렌더링 결과를 확인하니 `deployment.yaml`에 이미지가 하드코딩되어 있었음:

```bash
helm template ./helm/passorder-api  # 렌더링 결과 확인
# image: ...amazonaws.com/passorder-api:latest  ← 하드코딩
```

**해결**: `deployment.yaml`에서 Helm 템플릿 변수로 교체:

```yaml
image:{{ .Values.image.repository}}:{{ .Values.image.tag }}
```

**개선 / 배운 점**: Helm은 values.yaml을 자동으로 주입하지 않음. 템플릿에 명시적으로 변수를 참조해야 하며, `helm template` 명령으로 렌더링 결과를 사전에 검증하는 습관이 중요

---

## 온프레미스 vs AWS 비교

이 프로젝트 이전에 온프레미스 환경(VMware + GNS3)에서 동일한 아키텍처를 직접 구성했습니다.
직접 구성해본 경험 덕분에 AWS 관리형 서비스가 어떤 복잡성을 추상화하는지 이해할 수 있었습니다.

| 항목 | 온프레미스 | AWS |
| --- | --- | --- |
| 로드밸런서 | MetalLB 직접 설치 | ALB (관리형) |
| 네트워크 | Calico BGP 직접 설정 | VPC CNI |
| 게이트웨이 이중화 | HSRP 직접 구성 | Multi-AZ |
| 접근 제어 | Bastion Host + ACL + iptables | Security Group + IAM |
| TLS | cert-manager self-signed | ACM |
| 프로비저닝 | 수동 VM 설정 | Terraform |

---