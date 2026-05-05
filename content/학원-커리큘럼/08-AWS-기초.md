# AWS 클라우드 기술 기초 (TESS)

**학습 기간:** 2026.03.24 ~ 2026.03.25

---

### 1. 클라우드 개요

**온프레미스의 문제점**

| 문제 | 설명 |
| --- | --- |
| 높은 초기 비용 | 서비스 시작 전 서버·네트워크 장비 구매 필수 |
| 확장 어려움 | 서버 증설에 수 주~수 달 소요 |
| 자원 낭비 | 최대 트래픽 기준으로 서버 구매 → 평소엔 유휴 |

**클라우드의 핵심 특징**

| 특징 | 설명 |
| --- | --- |
| On-demand self service | 사용자가 직접 리소스 즉시 생성 |
| Rapid elasticity | 수요에 따라 빠르게 확장/축소 |
| Measured service | 사용한 만큼만 비용 지불 (Pay as you go) |

**클라우드 서비스 모델**

| 모델 | 사용자 관리 범위 | 예시 |
| --- | --- | --- |
| IaaS | OS + Middleware + App | EC2, Azure VM |
| PaaS | App만 개발 | Elastic Beanstalk |
| SaaS | 사용만 | Microsoft 365, Gmail |

**클라우드 배포 모델**

| 모델 | 특징 |
| --- | --- |
| Public Cloud | CSP가 인프라 운영, 인터넷으로 접근 |
| Private Cloud | 기업이 자체 클라우드 구축 (보안↑, 비용↑) |
| Hybrid Cloud | 온프레미스 + 클라우드 혼합 (VPN/DX 연결) |
| Multi Cloud | AWS + Azure 등 2개 이상 CSP 동시 사용 |

---

### 2. AWS Global Infrastructure

**Region → AZ → Edge Location**

| 단위 | 설명 |
| --- | --- |
| Region | 지리적으로 분리된 독립 데이터센터 집합 (ap-northeast-2 = 서울) |
| AZ (Availability Zone) | 리전 내 물리적으로 분리된 데이터센터 (ap-northeast-2a/b/c/d) |
| Edge Location | CloudFront CDN 캐시 서버, 전 세계 분산 |

> 고가용성 설계의 기본 = **Multi-AZ 배포**
> 

---

### 3. VPC (Virtual Private Cloud)

AWS 내 논리적으로 격리된 사설 네트워크.

**핵심 구성 요소**

| 구성요소 | 역할 |
| --- | --- |
| VPC | 격리된 네트워크 공간 (CIDR 블록 지정) |
| Subnet | VPC를 AZ 단위로 분할, Public/Private 구분 |
| Route Table | 서브넷의 트래픽 라우팅 규칙 |
| IGW (Internet Gateway) | VPC ↔ 인터넷 연결 |
| NAT Gateway | Private 서브넷 → 인터넷 단방향 허용 |
| Security Group | 인스턴스 레벨 방화벽 (Stateful) |
| NACL | 서브넷 레벨 방화벽 (Stateless) |

**Public vs Private Subnet**

| 구분 | Public Subnet | Private Subnet |
| --- | --- | --- |
| 인터넷 접근 | IGW로 직접 가능 | NAT Gateway 경유 |
| 주요 리소스 | ALB, Bastion Host | EC2, RDS, EKS |
| Route Table | 0.0.0.0/0 → IGW | 0.0.0.0/0 → NAT |

**Security Group vs NACL**

| 항목 | Security Group | NACL |
| --- | --- | --- |
| 적용 범위 | 인스턴스 | 서브넷 |
| 상태 | Stateful | Stateless |
| 규칙 | 허용만 | 허용 + 거부 |
| 기본값 | 모든 인바운드 거부 | 모든 트래픽 허용 |

---

### 4. EC2 (Elastic Compute Cloud)

**인스턴스 타입 (패밀리)**

| 패밀리 | 용도 |
| --- | --- |
| t (버스터블) | 개발/테스트, 낮은 기본 CPU |
| m (범용) | 균형잡힌 CPU/메모리 |
| c (컴퓨팅 최적화) | 고성능 CPU |
| r (메모리 최적화) | 인메모리 DB, 캐시 |
| p/g (가속 컴퓨팅) | GPU, ML |

**구매 옵션**

| 옵션 | 특징 | 비용 절감 |
| --- | --- | --- |
| On-Demand | 언제든 시작/중지 | 기준 |
| Reserved | 1~3년 약정 | 최대 72% 절감 |
| Spot | 잉여 용량 활용, 중단 가능 | 최대 90% 절감 |
| Savings Plans | 유연한 약정 (시간당 사용량) | 최대 66% 절감 |

**스토리지**

| 종류 | 특징 |
| --- | --- |
| EBS | 블록 스토리지, 인스턴스에 연결, 데이터 영구 보존 |
| Instance Store | 임시 스토리지, 인스턴스 중지 시 삭제 |
| EFS | 관리형 NFS, 여러 EC2 동시 마운트 가능 |

---

### 5. ELB / Auto Scaling

**ELB 종류**

| 종류 | L4/L7 | 용도 |
| --- | --- | --- |
| ALB | L7 | HTTP/HTTPS, 경로·호스트 기반 라우팅 |
| NLB | L4 | TCP/UDP, 초고성능, 고정 IP |
| GWLB | L3 | 서드파티 가상 어플라이언스 연동 |

**Auto Scaling 정책**

| 정책 | 설명 |
| --- | --- |
| Target Tracking | 목표 지표 유지 (CPU 60% 유지) |
| Step Scaling | 지표 범위별 단계적 조정 |
| Scheduled | 예정된 시간에 스케일링 |

---

### 6. S3 (Simple Storage Service)

- 객체 스토리지, 무제한 용량, 99.999999999% (11 nine) 내구성
- URL 기반 접근, 버킷 + 객체 키 구조

**스토리지 클래스**

| 클래스 | 용도 |
| --- | --- |
| Standard | 자주 접근하는 데이터 |
| Standard-IA | 자주 접근하지 않는 데이터 |
| Glacier | 아카이브 (검색 시간 분~시간) |
| Glacier Deep Archive | 장기 보관 (검색 시간 12시간) |
| Intelligent-Tiering | 접근 패턴에 따라 자동 이동 |

---

### 7. IAM (Identity and Access Management)

**핵심 개념**

| 개념 | 설명 |
| --- | --- |
| User | 개별 사람/서비스 계정 |
| Group | User 집합, 정책 일괄 적용 |
| Role | AWS 서비스나 사용자에게 임시 권한 부여 |
| Policy | JSON 형식 권한 정의 문서 |

**Policy 유형**

| 유형 | 설명 |
| --- | --- |
| Identity-based | User/Group/Role에 연결 |
| Resource-based | S3 버킷 정책 등 리소스에 연결 |
| SCP | AWS Organizations 단위 경계 정책 |

**AssumeRole / STS**
EC2, Lambda 등 AWS 서비스가 다른 서비스에 접근할 때 IAM Role을 임시 위임받아 사용. STS(Security Token Service)가 임시 자격증명 발급.

**보안 모범 사례**

- Root 계정 MFA 필수, 일반 작업에 사용 금지
- 최소 권한 원칙 (Least Privilege)
- Access Key 대신 IAM Role 사용

---

### 8. 주요 실습 내용

- VPC 생성 + Public/Private Subnet 구성
- Bastion Host를 통한 Private 서버 접속
- ELB + Auto Scaling 웹 서비스 구성
- EBS Volume 생성 및 연결
- S3 버킷 생성 및 정적 웹사이트 호스팅
- Aurora PostgreSQL 생성 및 Multi-AZ 고가용성
- VPC Interface Endpoint + SSM Session Manager

---