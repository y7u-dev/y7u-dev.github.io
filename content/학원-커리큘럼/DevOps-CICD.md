# DevOps 환경에서의 CI/CD

**학습 기간:** 2026.04.08 ~ 2026.04.12

---

### 1. DevOps와 CI/CD 개요

**DevOps란?**
개발(Development)과 운영(Operations)의 경계를 허물고, 협업·자동화·지속적 개선을 통해 소프트웨어를 빠르고 안정적으로 제공하는 문화·방법론.

**기존 방식의 문제**

- 개발팀이 코드 완성 → 운영팀에 전달 → 환경 불일치로 장애 발생
- 수동 배포 → 휴먼 에러, 환경 재현 불가
- 배포 주기 길어짐 → 피드백 지연

**DevOps가 해결하는 것**

| 문제 | 해결 |
| --- | --- |
| 환경 불일치 | 컨테이너(Docker)로 환경 통일 |
| 수동 배포 | CI/CD 파이프라인 자동화 |
| 느린 피드백 | 자동 테스트로 조기 오류 감지 |
| 수동 배포 일관성 부재 | GitOps로 선언적 배포 |

**CI / CD 구분**

| 개념 | 의미 |
| --- | --- |
| CI (Continuous Integration) | 코드 변경 시 자동으로 빌드·테스트·통합 |
| CD (Continuous Delivery) | 언제든 배포 가능한 상태 유지 (수동 승인 포함) |
| CD (Continuous Deployment) | 테스트 통과 시 운영 환경 자동 배포 |

---

### 2. CI/CD 파이프라인 구조

**파이프라인 기본 단계**

```
Source → Build → Test → Package → Publish → Deploy → Verify
```

| 단계 | 설명 | 산출물 |
| --- | --- | --- |
| Source | 저장소에서 코드 체크아웃 | 소스코드 작업 디렉터리 |
| Build | 컴파일, 의존성 설치 | 빌드 결과물 (.jar, .war 등) |
| Test | 단위·통합 테스트 자동 실행 | 테스트 결과 리포트 |
| Package | 컨테이너 이미지 빌드 | Docker Image |
| Publish | 레지스트리에 이미지 푸시 | ECR/Docker Hub의 이미지 |
| Deploy | 배포 환경에 적용 | 실행 중인 서비스 |
| Verify | 배포 후 정상 동작 확인 | 모니터링/알림 |

**배포 전략**

| 전략 | 설명 | 특징 |
| --- | --- | --- |
| In-place | 기존 서버에 직접 배포 | 빠름, 롤백 어려움 |
| Rolling | 인스턴스 순차 교체 | 무중단, 속도 조절 가능 |
| Blue/Green | 동일 환경 2벌 운영, 트래픽 전환 | 즉시 롤백 가능, 비용 2배 |
| Canary | 일부 트래픽만 신버전으로 점진적 전환 | 위험 최소화, 실시간 모니터링 |

**브랜치 전략**

| 전략 | 특징 |
| --- | --- |
| Git Flow | main/develop/feature/release/hotfix 브랜치 체계 |
| GitHub Flow | main + feature 브랜치, PR 기반 단순화 |
| Trunk Based | 단일 main 브랜치, 짧은 수명의 feature 브랜치 |

---

### 3. Jenkins 개요와 아키텍처

**Jenkins란?**
오픈소스 자동화 서버. 플러그인 기반으로 CI/CD 파이프라인 구성.

**아키텍처 (Master/Agent)**

```
Jenkins Controller (Master)
├── 파이프라인 스케줄링
├── 웹 UI 제공
└── Agent 관리

Agent (Node)
├── 실제 빌드·테스트 실행
└── Executor (병렬 작업 수 단위)
```

| 구성요소 | 역할 |
| --- | --- |
| Controller | 파이프라인 스케줄링, UI, 플러그인 관리 |
| Agent | 실제 작업 실행 환경 |
| Executor | Agent의 병렬 실행 슬롯 수 |
| Label | Agent에 붙이는 태그, Pipeline에서 실행 노드 선택에 사용 |

**Jenkins Job 유형**

| 유형 | 설명 |
| --- | --- |
| Freestyle | GUI 기반 단순 작업 |
| Pipeline | Jenkinsfile로 코드 기반 파이프라인 |
| Multibranch Pipeline | 브랜치별 자동 파이프라인 생성 |
| Multi-configuration | 여러 환경 조합 동시 실행 |

---

### 4. Jenkins Declarative Pipeline

**Jenkinsfile 기본 구조**

```groovy
pipeline {
    agent any               // 실행할 Agent 지정

    environment {           // 환경 변수
        APP_NAME = 'myapp'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }

    post {                  // 파이프라인 완료 후 처리
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
```

**주요 지시어**

| 지시어 | 설명 |
| --- | --- |
| `agent` | 실행 노드 지정 (any / label / docker) |
| `stages` | 단계 묶음 |
| `stage` | 개별 단계 정의 |
| `steps` | 실제 실행 명령 |
| `environment` | 환경 변수 선언 |
| `when` | 조건부 실행 |
| `parallel` | 병렬 실행 |
| `post` | 완료 후 처리 (always/success/failure) |

> ⚠️ **자주 헷갈리는 스펠링**
> 
> - `stages` (O) / `stage` (O) — 복수형 주의
> - `steps` (O) / `step` (X)
> - `Executor` (O) / `Excutor` (X)
> - `Multibranch` (O) / `Multibranch` 대소문자 주의
> - `Integration` (O) — CI의 I

---

### 5. GitHub Actions

**GitHub Actions란?**
GitHub 저장소에 내장된 CI/CD 플랫폼. `.github/workflows/` 에 YAML 파일로 정의.

**Workflow 기본 구조**

```yaml
name: CI Pipeline

on:                         # 트리거 조건
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:                    # Job 이름
    runs-on: ubuntu-latest  # 실행 환경 (Runner)

    steps:
      - uses: actions/checkout@v3         # 코드 체크아웃
      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Build
        run: mvn clean package
      - name: Test
        run: mvn test
```

**핵심 개념**

| 개념 | 설명 |
| --- | --- |
| Workflow | 자동화 전체 프로세스 (.yml 파일 단위) |
| Event (on) | Workflow 트리거 (push, PR, schedule 등) |
| Job | 독립 실행 단위, 병렬 실행 가능 |
| Step | Job 내 순차 실행 명령 |
| Action | 재사용 가능한 작업 단위 (`uses:`) |
| Runner | Job이 실행되는 환경 (GitHub 제공 또는 self-hosted) |

**주요 트리거**

```yaml
on:
  push:                     # 코드 푸시
  pull_request:             # PR 생성/수정
  schedule:                 # 정기 실행
    - cron: '0 9 * * 1'
  workflow_dispatch:        # 수동 실행
```

**Jenkins vs GitHub Actions**

| 항목 | Jenkins | GitHub Actions |
| --- | --- | --- |
| 서버 관리 | 직접 운영 | 불필요 (GitHub 제공) |
| 비용 | 서버 비용 | 무료 (공개 저장소) / 사용량 과금 |
| 설정 위치 | Jenkinsfile | .github/workflows/ |
| 커스터마이징 | 플러그인 기반, 매우 높음 | Actions 마켓플레이스 |

---

### 6. GitOps 개요

**GitOps란?**
Git 저장소를 **Single Source of Truth(단일 진실 공급원)** 로 삼아 인프라와 애플리케이션을 선언적으로 관리하는 운영 방식.

> ⚠️ "Single Point of Truth" (X) → "Single Source of Truth" (O)
> 

**GitOps 4대 원칙**

| 원칙 | 설명 |
| --- | --- |
| 선언적 | 원하는 상태를 코드로 선언 |
| 버전 관리 | 모든 변경사항이 Git에 기록 |
| 자동 적용 | Git 상태와 실제 상태 자동 동기화 |
| 지속적 검증 | 실제 상태가 Git 상태와 일치하는지 지속 확인 |

**Push 방식 vs Pull 방식**

| 방식 | 동작 | 보안 |
| --- | --- | --- |
| Push (기존 CI/CD) | CI 서버가 배포 환경에 직접 Push | 배포 환경 접근 권한 CI에 필요 |
| Pull (GitOps) | 클러스터 내 Agent가 Git 변경 감지 후 Pull | 클러스터 자격증명이 외부 노출 안 됨 |

---

### 7. Argo CD와 GitOps 운영

**Argo CD란?**
Kubernetes를 위한 GitOps 도구. Git 저장소의 매니페스트와 클러스터 실제 상태를 지속적으로 동기화.

```
Git 저장소 (원하는 상태)
        ↕ 감지·동기화
Argo CD
        ↕
Kubernetes 클러스터 (실제 상태)
```

**핵심 상태 2가지**

| 상태 | 설명 |
| --- | --- |
| **Sync Status** | Git 상태 ↔ 클러스터 상태 일치 여부 (Synced / OutOfSync) |
| **Health Status** | 애플리케이션 정상 동작 여부 (Healthy / Degraded / Progressing) |

> ⚠️ Sync Status ≠ Health Status
> 
> - Synced지만 Degraded 가능 (배포는 됐지만 Pod가 CrashLoop)
> - OutOfSync지만 Healthy 가능 (이전 버전이 정상 동작 중)

**Argo CD 주요 기능**

| 기능 | 설명 |
| --- | --- |
| Auto Sync | Git 변경 감지 시 자동 동기화 |
| Self Heal | 클러스터 상태가 변경되면 Git 상태로 자동 복구 |
| Rollback | 이전 Git 커밋 상태로 즉시 롤백 |
| Multi Cluster | 여러 클러스터 중앙 관리 |
| App of Apps | 여러 Application을 하나로 묶어 관리 |

**Kustomize vs Helm (Argo CD에서)**

| 도구 | 방식 | 특징 |
| --- | --- | --- |
| Kustomize | 오버레이 방식 | 베이스 + 환경별 패치 |
| Helm | 템플릿 + values | 패키지 관리, 버전 관리 |

---

### 8. 주요 실습 내용

| 실습 | 내용 |
| --- | --- |
| Jenkins CI 파이프라인 | Jenkinsfile 작성, ECR 이미지 빌드·푸시 |
| GitHub Actions CI | .github/workflows 작성, Docker 빌드·푸시 |
| Argo CD GitOps | Git 저장소 연결, Application 생성, Auto Sync 설정 |
| AWS 서비스 CI/CD | CodeCommit → CodeBuild → CodeDeploy → ECS 파이프라인 |

---