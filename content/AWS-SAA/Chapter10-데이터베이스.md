## 선택 기준

| 상황 | 서비스 |
| --- | --- |
| 복잡한 관계, SQL | RDS, Aurora |
| 초고속 조회, 유연한 스키마 | DynamoDB |
| 빠른 임시 저장 | ElastiCache |
| 대용량 분석 | Redshift |

## RDS

### Multi-AZ (고가용성)

```
[AZ-A] Primary ↔ 동기식 복제 ↔ [AZ-B] Standby
장애 시 Standby → Primary 자동 승격 (약 1~2분)
Standby는 읽기 트래픽 처리 안 함 (장애 대비 전용)
```

### Read Replica (읽기 확장)

```
[Primary] → 비동기 복제 → [Replica 1, 2, 3...]
쓰기: Primary / 읽기: Replica 분산
```

### RDS Proxy

Lambda ↔︎ RDS 사이 연결 풀 관리. Lambda가 수천 개 동시 실행돼도 DB 연결 수 안정.

> 서버리스 + RDS 조합이 나오면 → RDS Proxy
> 

## Aurora

AWS가 클라우드용으로 재설계. MySQL 대비 5배 빠름.

- 스토리지 자동 확장 (최대 128TB)
- 3개 AZ × 2 = **6개 복사본** 자동 유지 (스토리지 레이어)
- Replica들이 스토리지 공유 → 복제 지연 없음

```
[Primary 인스턴스]  ─┐
                     ├→ [공유 스토리지: 6개 복사본]
[Replica 인스턴스들] ─┘
```

> Replica 인스턴스 = 쿼리 처리하는 컴퓨팅 / 6개 복사본 = 데이터 저장 스토리지 (별개)
> 

## DynamoDB

완전 서버리스 NoSQL.

- 한 자릿수 ms 응답 속도
- 무제한 용량, 자동 확장

**구조:** 테이블 → 아이템 → Partition Key (필수) + Sort Key (선택) + 속성

**DynamoDB Streams:** 데이터 변경 이벤트 → Lambda 트리거 연결 가능

## 선택 요약

| 상황 | 서비스 |
| --- | --- |
| 복잡한 JOIN, 트랜잭션 | RDS |
| 고성능 관계형, 자동 확장 | Aurora |
| 초고속, 무한 확장, 서버리스 | DynamoDB |
| 간헐적 트래픽 관계형 DB | Aurora Serverless |