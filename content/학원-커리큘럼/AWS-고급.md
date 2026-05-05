# 모범사례기반 AWS 고급 아키텍처 설계

**학습 기간:** 2026.04.01 ~ 2026.04.07

---

### 1. 고급 멀티 티어 아키텍처와 설계 원칙

**MSA (Microservices Architecture)**
모놀리식 → MSA 전환으로 서비스별 독립 배포·확장 가능.

```
Monolith              MSA
─────────             ─────────────────────────
단일 서버              서비스 A → 독립 배포
모든 기능 결합  →      서비스 B → 독립 확장
장애 전파 큼           서비스 C → 독립 DB
```

**이벤트 기반 아키텍처**

| 패턴 | 구성 | 용도 |
| --- | --- | --- |
| Fan-out | SNS → SQS × N | 이벤트 복수 구독자 전달 |
| Queue 기반 | SQS → Lambda/Worker | 비동기 처리, 트래픽 완충 |
| Event Bridge | 이벤트 라우팅 허브 | 서비스 간 느슨한 결합 |

**SQS vs SNS**

| 항목 | SQS | SNS |
| --- | --- | --- |
| 방식 | 큐 (Pull) | 토픽 (Push) |
| 메시지 보존 | 최대 14일 | 전달 즉시 삭제 |
| 수신자 | 단일 Consumer | 다수 구독자 동시 전달 |
| 용도 | 비동기 처리, 완충 | 알림, Fan-out |

---

### 2. 멀티 계정 관리와 거버넌스

**Landing Zone**
AWS 모범사례 기반 멀티 계정 환경 자동 설정. AWS Control Tower로 구현.

**계정 분리 전략**

```
Root (Management Account)
├── Security OU
│   ├── Log Archive Account  (CloudTrail, Config 중앙 로그)
│   └── Audit Account        (보안 감사)
├── Infrastructure OU
│   └── Shared Services Account (DNS, 네트워크)
├── Dev OU
│   └── Dev Account
└── Prod OU
    └── Prod Account
```

**SCP 활용 예시**

- 특정 리전만 사용 허용
- Root 계정 사용 금지
- 특정 인스턴스 타입만 허용

**AWS Config Rules**

- 암호화되지 않은 EBS 탐지
- MFA 미설정 IAM User 탐지
- 퍼블릭 S3 버킷 탐지
- 규정 위반 시 자동 수정(Remediation) 가능

---

### 3. 고급 네트워크 연결 아키텍처

**Transit Gateway**
여러 VPC / 온프레미스를 허브-스포크 구조로 중앙 연결.

```
온프레미스 (VPN/DX)
        ↓
  Transit Gateway
   ↙    ↓    ↘
VPC-A  VPC-B  VPC-C
```

**VPC Peering vs Transit Gateway**

| 항목 | VPC Peering | Transit Gateway |
| --- | --- | --- |
| 연결 방식 | 1:1 | 허브-스포크 |
| 전이적 라우팅 | ❌ | ✅ |
| 관리 복잡도 | VPC 수 증가 시 급증 | 중앙 관리 |
| 대역폭 제한 | 없음 | 있음 (50Gbps) |

**Site-to-Site VPN**

```
온프레미스 CGW (Customer Gateway)
        ↕ IPsec 터널 (2개)
AWS VGW (Virtual Private Gateway) 또는 Transit Gateway
```

**Direct Connect (DX)**
전용 물리 회선으로 온프레미스 ↔ AWS 연결. VPN보다 안정적이고 대역폭 보장.

| 항목 | Site-to-Site VPN | Direct Connect |
| --- | --- | --- |
| 연결 방식 | 인터넷 + IPsec | 전용선 |
| 대역폭 | 제한적 | 1Gbps ~ 100Gbps |
| 비용 | 저렴 | 비쌈 |
| 구축 시간 | 즉시 | 수 주 |
| 안정성 | 인터넷 영향 받음 | 높음 |

**PrivateLink (VPC Endpoint Service)**
서비스 제공자 VPC → 소비자 VPC로 안전하게 서비스 노출. 인터넷 미경유.

---

### 4. 컨테이너 및 현대적 애플리케이션 플랫폼

**EKS 핵심 구성**

| 구성요소 | 설명 |
| --- | --- |
| EKS Control Plane | AWS가 완전 관리 (etcd, API Server 등) |
| Node Group | EC2 기반 Worker Node 그룹 |
| Fargate Profile | 서버리스 Pod 실행 |
| Pod Identity | IAM Role을 Pod에 직접 연결 (IRSA 대체) |

**EKS 스토리지 연동**

| CSI Driver | 스토리지 | 특징 |
| --- | --- | --- |
| EBS CSI | EBS | ReadWriteOnce, 단일 노드 |
| EFS CSI | EFS | ReadWriteMany, 다중 노드 공유 |

**ALB Ingress Controller**
AWS Load Balancer Controller로 Ingress → ALB 자동 생성. 경로/호스트 기반 라우팅.

**SQS 기반 비동기 아키텍처**

```
EC2/Lambda (Producer)
    → SQS Queue
        → EC2/Lambda (Consumer)
            → RDS / DynamoDB
```

트래픽 폭증 시 SQS가 버퍼 역할 → Consumer 자동 스케일링으로 처리.

---

### 5. 고급 보안 아키텍처

**WAF (Web Application Firewall)**
HTTP/HTTPS 레벨에서 악성 요청 차단. CloudFront·ALB·API Gateway에 연결.

| WAF 규칙 | 차단 대상 |
| --- | --- |
| AWS Managed Rules | OWASP Top 10, 알려진 악성 IP |
| Rate-based Rules | DDoS, 무차별 대입 공격 |
| Custom Rules | 특정 IP, 헤더, URI 패턴 |

**WAF 로그 분석**
WAF → Kinesis Firehose → S3 → Athena로 SQL 쿼리 분석.

**KMS (Key Management Service)**
암호화 키 중앙 관리. EBS·S3·RDS 등 대부분의 AWS 서비스와 통합.

| 키 유형 | 설명 |
| --- | --- |
| AWS Managed Key | AWS가 자동 관리, 무료 |
| Customer Managed Key (CMK) | 고객이 직접 관리, 세밀한 제어 |
| Custom Key Store | HSM 기반 키 저장 |

**Secrets Manager**
DB 비밀번호·API 키 등 민감 정보를 암호화 저장. 자동 로테이션 지원. KMS와 연동.

```
애플리케이션 → Secrets Manager API → 복호화된 시크릿 반환
(코드에 하드코딩 ❌)
```

**Shield**

| 등급 | 보호 대상 | 비용 |
| --- | --- | --- |
| Shield Standard | 모든 AWS 고객 자동 적용 | 무료 |
| Shield Advanced | DDoS 고급 보호, 비용 보상 | 유료 |

**GuardDuty**
AWS 환경의 위협 탐지 서비스. CloudTrail·VPC Flow Logs·DNS 로그 분석으로 악성 활동 감지.

---

### 6. 비용 최적화와 엣지 아키텍처

**비용 최적화 핵심 전략**

| 전략 | 방법 |
| --- | --- |
| 올바른 규모 조정 | 실제 사용량 분석 후 인스턴스 타입 최적화 |
| 구매 옵션 활용 | Reserved/Savings Plans로 약정 할인 |
| 불필요 리소스 제거 | 미사용 EBS, 오래된 스냅샷, 유휴 ELB 삭제 |
| 스토리지 최적화 | S3 수명 주기로 저렴한 클래스 자동 이동 |
| 서버리스 전환 | Lambda, Fargate로 유휴 시간 비용 제거 |

**AWS Cost Explorer**
비용 분석·예측·이상 탐지. 리소스별/서비스별/태그별 분석.

**CloudFront 고급 활용**

| 기능 | 설명 |
| --- | --- |
| Origin Shield | 오리진 앞단 추가 캐시 계층, 오리진 부하 감소 |
| Lambda@Edge | 엣지에서 코드 실행 (요청/응답 커스터마이징) |
| CloudFront Functions | 초경량 JS 함수, 엣지에서 URL 리다이렉트·헤더 조작 |
| Signed URL/Cookie | 프리미엄 콘텐츠 접근 제어 |

---

### 7. 주요 실습 내용

| 실습 | 내용 |
| --- | --- |
| Transit Gateway | VPC 3개 허브-스포크 연결 |
| Site-to-Site VPN | 온프레미스 ↔ AWS 1/2터널 구성 |
| Gateway/Interface Endpoint | S3 Gateway Endpoint, SSM Interface Endpoint |
| Route 53 Routing Policy | Weighted·Latency·Failover 정책 |
| AWS Backup | 백업 계획 생성 및 복구 |
| WAF | ALB에 WAF 연결, 규칙 설정, Athena 로그 분석 |
| KMS + Secrets Manager | CMK 생성, RDS 비밀번호 로테이션 |
| EKS | 클러스터 설치, ALB Ingress, EBS/EFS CSI, Pod Identity, SQS 연동 |
| ECR | 이미지 빌드 및 Push, EKS에서 Pull |

---