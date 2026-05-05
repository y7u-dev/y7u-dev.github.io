# Google 쿠버네티스 아키텍처 설계 및 구축

**학습 기간:** 

---

### 1. GKE 개요와 EKS 대비 운영 모델 비교

**GKE(Google Kubernetes Engine)란?**
Google Cloud의 관리형 Kubernetes 서비스. EKS와 마찬가지로 Control Plane을 Google이 관리

**GKE와 EKS의 공통점**

- 둘 다 관리형 Kubernetes (Control Plane 직접 관리 불필요)
- 클러스터 기반 운영 구조
- 클라우드 네트워크와 IAM 위에서 동작

**GKE가 EKS와 다른 핵심 차이**

| 항목 | EKS (AWS) | GKE (GCP) |
| --- | --- | --- |
| 운영 모드 | 단일 모드 | Standard / Autopilot 선택 |
| 버전 관리 | 수동 또는 관리형 노드그룹 | Release Channel 기반 자동 업그레이드 |
| 워크로드 권한 | IRSA (IAM Role for SA) | Workload Identity Federation for GKE |
| 다중 클러스터 | 별도 관리 도구 필요 | Fleet으로 통합 관리 |
| 서비스 노출 | Ingress + ALB Controller | Gateway API + GKE 관리형 LB |

---

### 2. Standard 모드와 Autopilot 모드

**운영 모드 비교**

| 항목 | Standard | Autopilot |
| --- | --- | --- |
| 노드 관리 | 사용자가 노드풀 직접 설계 | Google이 자동 관리 |
| 스케일링 | 수동 또는 Cluster Autoscaler 설정 | 자동 (Pod 요청 기반) |
| 보안 기본값 | 사용자가 설정 | 강화된 기본값 자동 적용 |
| 유연성 | 높음 (OS 설정 등 제어 가능) | 낮음 (일부 설정 제한) |
| 비용 모델 | 노드 단위 과금 | Pod 단위 과금 |
| Release Channel | 선택 사항 | 필수 |

```
Autopilot — 운영 자동화와 표준화에 강함
Standard  — 유연성과 세밀한 제어에 강함
```

**Release Channel**
GKE가 제공하는 버전 자동 관리 채널. 클러스터 업그레이드를 Google이 일정에 맞춰 관리

| 채널 | 특징 |
| --- | --- |
| Rapid | 최신 버전 빠르게 적용 |
| Regular | 균형 잡힌 업데이트 주기 (권장) |
| Stable | 검증 후 안정적으로 적용 |

---

### 3. GKE 클러스터 생성 시 필수 설계 요소

**클러스터 배치 방식**

| 유형 | 설명 | 특징 |
| --- | --- | --- |
| 존형(Zonal) | 단일 Zone에 배치 | 비용 낮음, HA 미제공 |
| 리전형(Regional) | 여러 Zone에 분산 배치 | Control Plane HA 제공, 운영 권장 |

**VPC-native 클러스터 (권장 네트워크 모델)**

GKE는 **VPC-native 클러스터**를 기본 권장
Pod IP가 GCP VPC CIDR에서 직접 할당되어 VPC 내에서 라우팅 가능

```
일반 클러스터: Pod IP ≠ VPC CIDR → 별도 오버레이 네트워크 필요
VPC-native:   Pod IP = VPC Secondary Range → VPC에서 직접 라우팅
```

**IP 대역 설계**

| 대역 | 설명 |
| --- | --- |
| Node CIDR | VM(노드)에 할당되는 IP |
| Pod CIDR | Pod에 할당되는 IP (Secondary Range) |
| Service CIDR | Service 가상 IP에 사용되는 대역 |

**Private Cluster**
Control Plane과 노드를 외부 인터넷에 노출하지 않는 구성
운영 환경 보안 강화 시 권장

---

### 4. Workload Identity Federation for GKE

**왜 필요한가?**
GKE Pod가 Cloud Storage, Secret Manager 등 Google Cloud API에 접근할 때 안전한 자격 부여 방식 필요

**나쁜 패턴 — 서비스 계정 키 파일 사용**

```
문제점:
- 키 파일이 유출되면 영구적 권한 노출
- 키 교체 관리 부담
- 비밀번호처럼 관리해야 하는 장기 자격증명
```

**권장 패턴 — Workload Identity Federation for GKE**

```
K8s ServiceAccount ↔ IAM Service Account 연결
    ↓
Pod → K8s ServiceAccount 토큰 발급
    ↓
Google이 토큰 검증 → 임시 자격증명 발급
    ↓
Cloud Storage / Secret Manager 등 접근
```

**EKS IRSA vs GKE Workload Identity 비교**

| 항목 | EKS IRSA | GKE Workload Identity |
| --- | --- | --- |
| 연결 방식 | K8s SA ↔︎ IAM Role (OIDC) | K8s SA ↔︎ IAM Service Account |
| 자격증명 | STS 임시 토큰 | Google 임시 토큰 |
| 설정 위치 | OIDC Provider + Role Trust Policy | Workload Identity Pool + SA Binding |
| Autopilot | - | 자동 활성화 |

---

### 5. GKE 서비스 노출과 Gateway API

**서비스 노출 방식 진화**

```
ClusterIP (내부 통신)
  → NodePort (노드 포트 직접 노출)
    → LoadBalancer Service (외부 IP 하나)
      → Ingress (L7 라우팅, 하나의 LB로 여러 서비스)
        → Gateway API (운영형 트래픽 설계 모델)
```

**Ingress의 한계**

- 역할이 하나의 리소스에 집중 (관리자, 개발자 역할 분리 어려움)
- 복잡한 트래픽 설정이 어노테이션에 의존
- 멀티 클러스터 구성에 불리

**Gateway API 핵심 리소스**

| 리소스 | 역할 |
| --- | --- |
| GatewayClass | 어떤 LB 구현체를 사용할지 정의 |
| Gateway | 트래픽 진입점 (포트, 프로토콜 정의) |
| HTTPRoute | 트래픽 라우팅 규칙 (경로, 헤더 기반) |

```
GKE에서 Gateway API = Google Cloud Load Balancer와 자동 연결
Ingress는 "도구", Gateway API는 "운영형 설계 모델"
```

**GKE Gateway API vs Ingress**

| 항목 | Ingress | Gateway API |
| --- | --- | --- |
| 역할 분리 | 하나의 리소스에 집중 | GatewayClass / Gateway / Route 분리 |
| 트래픽 제어 | 어노테이션 의존 | 선언적 정책으로 표현 |
| 멀티 클러스터 | 어려움 | 지원 |
| 관리 주체 | 단일 | 인프라팀 / 앱팀 분리 가능 |

---

### 6. 멀티클라우드 아키텍처 개요

**멀티클라우드란?**
둘 이상의 퍼블릭 클라우드 환경을 함께 사용하는 아키텍처 (AWS + GCP 등)

**AWS + GCP를 함께 쓰는 주요 시나리오**

- 기존 AWS 서비스 + 신규 GKE 도입 (단계적 전환)
- 리전/서비스별 최적화 (AI는 GCP, 기존 앱은 AWS)
- 벤더 종속성 방지

**멀티클라우드 설계 고려 영역**

| 영역 | 핵심 내용 |
| --- | --- |
| 네트워크 | HA VPN + Cloud Router로 사설 경로 연결 |
| 인증/권한 | IAM 경계 설계, Workload Identity 활용 |
| 관찰성 | 중앙 모니터링 통합 |
| 보안 경계 | 방화벽/SG 규칙 명확화 |

---

### 7. GKE와 AWS 리소스 간 통신 시나리오

**HA VPN으로 연결했을 때 되는 것 vs 안 되는 것**

```
HA VPN이 만들어주는 것:
  GCP VPC CIDR ↔ AWS VPC CIDR 간 사설 경로
  BGP 기반 동적 경로 교환

VPN만으로 해결되지 않는 것:
  - 어떤 대역을 실제로 광고할지
  - 방화벽/보안그룹 허용 여부
  - DNS 이름 해석
  - GKE 내부 Service 접근 방식
  - Pod 대역 vs Node 대역 선택
```

**GKE IP 대역과 접근 방식**

| 대상 | 접근 방법 |
| --- | --- |
| Node IP | VPN 경로만 있으면 직접 접근 (단순) |
| Pod IP | VPC-native이면 가능, 아니면 오버레이 추가 필요 |
| Service IP | 가상 IP라 VPN으로 직접 불가 → Internal LB 또는 DNS 필요 |

**AWS → GKE 서비스 접근 권장 패턴**

```
AWS EC2/EKS
  └→ HA VPN (사설 경로)
      └→ GCP Internal Load Balancer (Service 앞단)
          └→ GKE Service → Pod
```

---

### 8. 핵심 개념 정리

| 개념 | 한 줄 요약 |
| --- | --- |
| GKE | Google Cloud 관리형 Kubernetes |
| Standard 모드 | 노드 직접 관리, 유연성 높음 |
| Autopilot 모드 | Google이 노드 관리, 운영 자동화 |
| Release Channel | GKE 버전 자동 업그레이드 채널 |
| VPC-native 클러스터 | Pod IP를 VPC CIDR에서 직접 할당 |
| Workload Identity | K8s SA ↔︎ GCP IAM SA 연결, 키 파일 불필요 |
| Gateway API | Ingress를 대체하는 운영형 트래픽 설계 모델 |
| Fleet | 다중 GKE 클러스터 통합 관리 |
| HA VPN | AWS-GCP 간 고가용성 사설 연결 |
| Cloud Router | BGP 기반 동적 경로 교환 |