# Chapter11_캐싱

# Chapter 11. 캐싱 전략

## ElastiCache

| 구분 | Redis | Memcached |
| --- | --- | --- |
| 영속성 | 있음 | 없음 |
| 자료구조 | 다양 | 단순 String |
| 용도 | 세션, 리더보드, 실시간 분석 | 단순 객체 캐싱 |

## 캐싱 패턴

**Lazy Loading:** Cache Miss → DB 조회 → 캐시 저장 → 반환
**Write Through:** DB 저장 + 캐시 동시 저장
**TTL:** 만료 시간 설정으로 오래된 데이터 방지

## DAX (DynamoDB Accelerator)

DynamoDB 전용 캐시. 코드 변경 없이 적용. ms → 마이크로초(μs).

| 상황 | 서비스 |
| --- | --- |
| DynamoDB 캐싱, 코드 변경 싫음 | DAX |
| RDS 캐싱, 복잡한 캐싱 로직 | ElastiCache |

## ElastiCache for Redis 고가용성

```
다중 AZ Redis 복제 그룹:
├── Primary 클러스터 (AZ-A)
└── Read Replica들 (AZ-B, AZ-C...)

Primary 장애 시 Replica가 자동 승격
샤드로 데이터 분산 → 확장성 향상
```