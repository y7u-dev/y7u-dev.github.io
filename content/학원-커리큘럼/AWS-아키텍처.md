# AWS 아키텍처 설계 (ARC)

**학습 기간:** 2026.03.26 ~2026.03.31

---

### 1. AWS 아키텍처 설계 기본

**Well-Architected Framework 6대 원칙**

| 원칙 | 핵심 내용 |
| --- | --- |
| 운영 우수성 | 코드로 운영, 자동화, 지속적 개선 |
| 보안 | 최소 권한, 모든 계층 방어, 추적 가능성 |
| 안정성 | 장애 자동 복구, 수평 확장, 용량 관리 |
| 성능 효율성 | 적절한 리소스 선택, 서버리스 활용 |
| 비용 최적화 | 불필요한 비용 제거, 사용량 분석 |
| 지속 가능성 | 에너지 효율 극대화 |

**RPO vs RTO**

| 개념 | 설명 |
| --- | --- |
| RPO (Recovery Point Objective) | 장애 시 허용 가능한 **데이터 손실 시점** (백업 주기 기준) |
| RTO (Recovery Time Objective) | 장애 시 허용 가능한 **서비스 복구 시간** |

> RPO가 낮을수록 → 더 자주 백업 / RTO가 낮을수록 → 더 빠른 복구 체계 필요
> 

---

### 2. AWS 계정 보안 아키텍처

**AWS Organizations**
여러 AWS 계정을 중앙에서 관리하는 서비스.

```
Root
└── Management Account
    ├── OU (Dev)
    │   ├── Account A
    │   └── Account B
    └── OU (Prod)
        └── Account C
```

**SCP (Service Control Policy)**
OU/계정 단위로 사용 가능한 서비스/액션을 제한하는 경계 정책. IAM 정책보다 상위 적용.

> SCP는 허용할 수 있는 최대 권한을 정의. IAM 정책이 허용해도 SCP가 막으면 불가.
> 

**AWS Config**
리소스 변경 이력 추적 및 규정 준수 자동 평가.

**CloudTrail**
AWS API 호출 로그 기록. 보안 감사·이상 탐지에 필수.

---

### 3. VPC 기반 아키텍처 설계

**3-Tier VPC 구조**

```
Internet
    ↓
IGW
    ↓
Public Subnet (ALB, Bastion)
    ↓
Private Subnet (EC2, EKS)
    ↓
DB Subnet (RDS, ElastiCache)
```

**VPC 간 연결 방법**

| 방법 | 특징 |
| --- | --- |
| VPC Peering | 1:1 연결, 전이적 라우팅 불가 |
| Transit Gateway | 허브-스포크 구조, 다수 VPC 중앙 연결 |
| PrivateLink | 서비스 제공자 → 소비자 단방향 안전 연결 |
| VPN (Site-to-Site) | 온프레미스 ↔ AWS 암호화 연결 |
| Direct Connect (DX) | 온프레미스 ↔ AWS 전용선 연결 |

**VPC Endpoint**

| 종류 | 방식 | 용도 |
| --- | --- | --- |
| Gateway Endpoint | 라우팅 테이블 기반 | S3, DynamoDB |
| Interface Endpoint | ENI 기반 (PrivateLink) | 대부분의 AWS 서비스 |

---

### 4. EC2 컴퓨팅 아키텍처

**배치 그룹 (Placement Group)**

| 종류 | 특징 | 사용 사례 |
| --- | --- | --- |
| Cluster | 동일 AZ, 낮은 지연시간 | HPC, 빅데이터 |
| Spread | 다른 하드웨어에 분산 | 소수의 고가용성 인스턴스 |
| Partition | 파티션 단위 분산 | Hadoop, Cassandra |

**EBS 볼륨 타입**

| 타입 | 특징 | 용도 |
| --- | --- | --- |
| gp3 | 범용 SSD, 비용 효율 | 대부분의 워크로드 |
| io2 | 고성능 SSD, 높은 IOPS | 고성능 DB |
| st1 | 처리량 최적화 HDD | 빅데이터, 로그 |
| sc1 | 콜드 HDD | 접근 빈도 낮은 데이터 |

---

### 5. 스토리지 아키텍처

**S3 핵심 기능**

| 기능 | 설명 |
| --- | --- |
| 버전 관리 | 객체 이전 버전 보존, 실수 복구 |
| 수명 주기 정책 | 스토리지 클래스 자동 전환 / 만료 삭제 |
| 복제 | CRR(교차 리전) / SRR(동일 리전) |
| 이벤트 알림 | 업로드 시 Lambda/SQS/SNS 트리거 |
| Pre-signed URL | 임시 접근 URL 생성 |

**EFS vs EBS vs S3**

| 항목 | EFS | EBS | S3 |
| --- | --- | --- | --- |
| 타입 | 파일 시스템 | 블록 | 객체 |
| 동시 접근 | 다중 EC2 | 단일 EC2 | 무제한 |
| 프로토콜 | NFS | - | HTTP |
| 용도 | 공유 파일 | OS/DB 디스크 | 정적 파일/백업 |

---

### 6. 데이터베이스 아키텍처

**RDS**
관리형 관계형 DB. Multi-AZ로 자동 Failover, Read Replica로 읽기 분산.

**Aurora**
AWS 독자 개발 DB 엔진 (MySQL/PostgreSQL 호환). RDS 대비 최대 5배(MySQL) / 3배(PostgreSQL) 성능. 스토리지 3개 AZ에 6개 복사본 자동 유지.

**ElastiCache**
인메모리 캐시 서비스.

| 엔진 | 특징 |
| --- | --- |
| Redis | 자료구조 다양, 복제/클러스터 지원, 영속성 옵션 |
| Memcached | 단순 캐시, 멀티스레드, 자동 샤딩 |

**DynamoDB**
완전관리형 NoSQL. Partition Key 설계가 성능/비용 좌우.

---

### 7. 모니터링 및 자동화

**CloudWatch**
AWS 리소스 메트릭 수집·알람·대시보드.

| 기능 | 설명 |
| --- | --- |
| Metrics | CPU, 네트워크 등 리소스 수치 |
| Logs | 로그 수집 및 쿼리 (CloudWatch Logs Insights) |
| Alarms | 임계값 초과 시 알림/Auto Scaling 트리거 |
| Events/EventBridge | 이벤트 기반 자동화 |

**CloudFormation**
YAML/JSON 템플릿으로 인프라 코드화 (IaC). 스택 단위 관리, 드리프트 감지.

**Elastic Beanstalk**
코드만 업로드하면 EC2·ELB·Auto Scaling 자동 구성. PaaS 방식.

---

### 8. 컨테이너 아키텍처

| 서비스 | 설명 |
| --- | --- |
| ECR | 컨테이너 이미지 레지스트리 |
| ECS | AWS 관리형 컨테이너 오케스트레이션 |
| EKS | AWS 관리형 Kubernetes |
| Fargate | 서버리스 컨테이너 실행 (EC2 관리 불필요) |

**ECS vs EKS vs Fargate**

| 항목 | ECS | EKS | Fargate |
| --- | --- | --- | --- |
| 오케스트레이션 | AWS 독자 | Kubernetes | ECS/EKS 위에서 실행 |
| 노드 관리 | 필요 (EC2 모드) | 필요 | 불필요 |
| K8s 호환 | ❌ | ✅ | △ |

---

### 9. 서버리스 아키텍처

**Lambda**
코드만 배포, 서버 관리 불필요. 요청 기반 과금, 최대 실행 시간 15분.

**API Gateway + Lambda 패턴**

```
Client → API Gateway → Lambda → DynamoDB
```

완전 서버리스 REST API 구성.

**Fan-out 패턴 (SNS + SQS)**

```
Producer → SNS Topic → SQS Queue 1 → Consumer 1
                     → SQS Queue 2 → Consumer 2
                     → Lambda
```

하나의 이벤트를 여러 서비스에 동시 전달.

---

### 10. 엣지 아키텍처

**CloudFront**
글로벌 CDN. 엣지 로케이션에서 콘텐츠 캐시 제공 → 지연시간 감소, 오리진 부하 감소.

| 오리진 | 설명 |
| --- | --- |
| S3 | 정적 웹사이트, 파일 배포 |
| ALB | 동적 컨텐츠 |
| EC2 | 직접 연결 |

**Route 53**
AWS 관리형 DNS 서비스.

| 라우팅 정책 | 용도 |
| --- | --- |
| Simple | 단순 레코드 |
| Weighted | 비율 기반 트래픽 분산 |
| Latency | 지연시간 기반 |
| Failover | 헬스체크 기반 장애 조치 |
| Geolocation | 사용자 위치 기반 |

---

### 11. 백업 및 복구 아키텍처

**재해 복구 전략 (비용↑ = RTO/RPO↓)**

| 전략 | RTO | RPO | 특징 |
| --- | --- | --- | --- |
| Backup & Restore | 시간 | 시간 | 가장 저렴, 복구 느림 |
| Pilot Light | 분~시간 | 분 | 핵심 시스템만 최소 운영 |
| Warm Standby | 분 | 초 | 축소 규모로 상시 운영 |
| Multi-Site Active/Active | 실시간 | 실시간 | 가장 비쌈, 즉시 전환 |

**AWS Backup**
여러 서비스(EBS, RDS, EFS 등)의 백업을 중앙에서 정책으로 관리.

---