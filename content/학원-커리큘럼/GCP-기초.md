# Google 클라우드 핵심 서비스

**학습 기간:** 

---

### 1. GCP 개요와 AWS와의 감각 차이

**GCP(Google Cloud Platform)란?**
Google이 자사 서비스를 운영하면서 축적한 인프라 기술을 외부에 제공하는 클라우드 서비스

**AWS vs GCP 설계 철학 차이**

| 항목 | AWS | GCP |
| --- | --- | --- |
| 운영 단위 | Account 중심 | Project 중심 |
| 네트워크 | Region 단위 VPC | 글로벌 VPC (리전 무관) |
| 서비스 구성 | 세밀하게 분화된 서비스 조합 | 글로벌 구조 위에서 일관된 자원 운영 |
| 계층 구조 | Organizations / OU | Organization / Folder / Project |

**GCP 글로벌 인프라 구조**

| 단위 | 설명 |
| --- | --- |
| Region | 지리적으로 분리된 데이터센터 집합 (서울: asia-northeast3) |
| Zone | Region 내 독립적 물리 위치. VM은 Zone에 배치 |
| Multi-region | 여러 Region에 걸친 광역 저장 단위 (Cloud Storage 등) |

**GCP 리소스 계층 구조**

```
Organization (회사 전체)
  └── Folder (부서/환경 단위)
        └── Project (운영 단위)
              └── Resources (VM, 버킷, DB 등)
```

- **Project가 핵심 운영 단위** — 모든 리소스는 Project에 속함
- 상위 계층의 IAM 정책은 하위로 상속됨
- AWS Account ↔︎ GCP Project 감각으로 이해

---

### 2. IAM (Identity and Access Management)

**GCP IAM 핵심 구성 요소**

| 개념 | 설명 | AWS 대응 |
| --- | --- | --- |
| Principal | 권한을 받는 주체 (사람, SA 등) | User / Role |
| Permission | 개별 API 작업 허용 단위 | Action |
| Role | Permission 묶음 | IAM Policy |
| Policy Binding | Principal + Role + Resource 범위 연결 | Role 연결 |

**Role 종류**

| Role 유형 | 설명 |
| --- | --- |
| Basic Role | Viewer / Editor / Owner (너무 넓어 운영 비권장) |
| Predefined Role | GCP가 서비스별로 미리 정의한 역할 |
| Custom Role | 필요한 Permission만 골라서 만든 역할 |

**Service Account**
사람이 아니라 **워크로드(VM, Pod 등)에 부여하는 자격 계정**
AWS IAM Role for EC2 / IRSA와 유사한 개념

```
GCP에서는 서비스 계정 키(.json)를 직접 사용하는 것보다
Workload Identity Federation 방식을 권장
```

---

### 3. VPC, Subnet, Firewall

**GCP VPC의 핵심 특징**

- **VPC는 글로벌 리소스** — 하나의 VPC 안에 여러 리전의 서브넷 배치 가능
- AWS는 리전마다 VPC를 따로 만들지만 GCP는 하나의 VPC가 여러 리전을 포괄

**VPC 모드 비교**

| 모드 | 특징 |
| --- | --- |
| Auto mode VPC | 자동으로 각 리전에 서브넷 생성. 빠르지만 CIDR 제어 어려움 |
| Custom mode VPC | 서브넷을 직접 설계. 운영 환경 권장 |

**Firewall Rules**
AWS Security Group과 유사하지만 VPC 레벨에서 관리
- Ingress / Egress 규칙 분리
- 태그(target tag) 기반으로 특정 VM에만 적용 가능

**GCP VPC vs AWS VPC**

| 항목 | AWS VPC | GCP VPC |
| --- | --- | --- |
| 범위 | 리전 단위 | 글로벌 |
| 서브넷 | 리전 VPC 내 AZ 단위 | 특정 리전에 배치 |
| 라우팅 | 라우팅 테이블 직접 관리 | 기본 라우팅 자동 관리 |
| 방화벽 | Security Group (인스턴스) + NACL (서브넷) | Firewall Rules (VPC 레벨, 태그 기반) |

---

### 4. Compute Engine

**Compute Engine이란?**
AWS EC2와 대응되는 GCP의 IaaS 컴퓨팅 서비스. OS 수준 제어 가능

**머신 타입 구조**

```
머신 패밀리 (General Purpose, Compute Optimized 등)
  └── 머신 시리즈 (E2, N2, C2 등)
        └── 머신 타입 (e2-medium, n2-standard-4 등)
```

**AWS EC2 vs GCP Compute Engine**

| 항목 | AWS EC2 | GCP Compute Engine |
| --- | --- | --- |
| 타입 체계 | 인스턴스 타입 (t3.medium 등) | 머신 패밀리/시리즈/타입 |
| 스토리지 | EBS | Persistent Disk |
| 초기화 | User Data | Startup Script (메타데이터 기반) |
| 권한 부여 | IAM Role (Instance Profile) | Service Account 연결 |

**Startup Script**
VM 부팅 시 자동 실행되는 초기화 스크립트. AWS User Data와 동일 역할
인스턴스 메타데이터를 기반으로 게스트 에이전트가 실행

---

### 5. Cloud Storage

**Cloud Storage란?**
AWS S3와 직접 대응되는 객체 스토리지 서비스

**Storage Class 비교**

| 클래스 | 용도 | AWS 대응 |
| --- | --- | --- |
| Standard | 자주 접근하는 데이터 | S3 Standard |
| Nearline | 월 1회 미만 접근 | S3 Standard-IA |
| Coldline | 분기 1회 미만 접근 | S3 Glacier |
| Archive | 1년 1회 미만 접근 | S3 Glacier Deep Archive |

**AWS S3 vs GCP Cloud Storage 차이점**

- GCP는 **Uniform bucket-level access** 권장 — ACL 대신 IAM 중심 단순화
- CLI는 기존 `gsutil` → 현재는 `gcloud storage` 권장
- 버킷 이름은 전 세계에서 고유해야 함

---

### 6. Cloud SQL

**Cloud SQL이란?**
AWS RDS와 대응되는 관리형 관계형 데이터베이스 서비스
MySQL, PostgreSQL, SQL Server 지원

**연결 방식**

| 방식 | 설명 | 특징 |
| --- | --- | --- |
| Public IP | 인터넷 IP로 직접 연결 | 허용 네트워크 명시 필요 |
| Private IP | VPC 사설 연결 | 보안 권장, VPC Peering 필요 |
| Auth Proxy | Cloud SQL Auth Proxy를 통한 안전한 연결 | IAM 기반 인증, 암호화 자동 |

**AWS RDS vs GCP Cloud SQL 차이점**

- GCP는 DB 연결 방식을 네트워크 + 프록시 관점에서 더 명확하게 설계
- `SUPER` 권한 등 일부 고급 DB 권한은 제한됨
- Private IP 사용 시 VPC 사설 연결 설계 필요

---

### 7. Load Balancing과 확장 구조

**GCP Load Balancer 유형**

| 유형 | 계층 | 범위 | AWS 대응 |
| --- | --- | --- | --- |
| Global External Application LB | L7 HTTP/HTTPS | 글로벌 | ALB (글로벌) |
| Regional External Application LB | L7 HTTP/HTTPS | 리전 | ALB (리전) |
| External Network LB | L4 TCP/UDP | 리전 | NLB |
| Internal Application LB | L7 내부 | 리전 | Internal ALB |

**GCP 자동 확장 구조**

```
Instance Template (VM 표준 사양 정의)
  └── Managed Instance Group (MIG) — VM 그룹 관리
        ├── Autoscaling (CPU/LB 기반 자동 확장)
        ├── Autohealing (비정상 인스턴스 자동 교체)
        └── Health Check
```

- MIG = AWS Auto Scaling Group 감각
- Instance Template = AWS Launch Template 감각

---

### 8. Monitoring & Logging

**Cloud Monitoring**
AWS CloudWatch Metrics/Dashboard 대응
CPU, 디스크, 네트워크, 응답시간 등 수치 지표 수집 및 알람

**Cloud Logging**
AWS CloudWatch Logs / Logs Insights 대응
시스템·애플리케이션·감사 로그 수집 및 검색

**AWS vs GCP 모니터링 대응**

| AWS | GCP |
| --- | --- |
| CloudWatch Metrics | Cloud Monitoring |
| CloudWatch Logs | Cloud Logging |
| CloudWatch Logs Insights | Log Explorer (쿼리 기반) |
| CloudWatch Alarms | Alerting Policy |
| CloudWatch Dashboard | Monitoring Dashboard |

---

### 9. 핵심 서비스 AWS ↔︎ GCP 대응표

| AWS | GCP | 설명 |
| --- | --- | --- |
| Account | Project | 기본 운영 단위 |
| EC2 | Compute Engine | 가상머신 |
| S3 | Cloud Storage | 객체 스토리지 |
| RDS | Cloud SQL | 관리형 RDB |
| VPC | VPC | 가상 네트워크 (GCP는 글로벌) |
| Security Group | Firewall Rules | 인스턴스 트래픽 제어 |
| IAM Role | Service Account | 워크로드 권한 부여 |
| ALB | Application Load Balancer | L7 로드밸런서 |
| Auto Scaling Group | Managed Instance Group | VM 자동 확장 |
| CloudWatch | Cloud Monitoring + Logging | 모니터링/로그 |
| IRSA | Workload Identity Federation | Pod 권한 부여 |