## DMS (Database Migration Service)

**무중단 마이그레이션:**
1. 서비스 운영 중 전체 데이터 초기 복사
2. CDC(Change Data Capture)로 변경사항 실시간 동기화
3. 완전 동기화 후 짧은 순간에 전환

- 동종 (MySQL → RDS MySQL): DMS만으로 가능
- 이종 (Oracle → Aurora): SCT로 스키마 변환 후 DMS

## Snow Family

| 서비스 | 용량 | 용도 |
| --- | --- | --- |
| Snowcone | 8TB | 오지, 드론 운반 |
| Snowball Edge Storage | 80TB | 대용량 데이터 이전 |
| Snowball Edge Compute | 42TB + 컴퓨팅 | 오지에서 처리 후 전송 |
| Snowmobile | 100PB | 데이터센터 전체 이전 |

**Snowball 선택 신호:**
- “네트워크 대역폭 최소화” → Snowball ✅
- “고속 인터넷 있음” → Snowball ❌ (Transfer Acceleration 사용)

## DataSync vs S3 파일 게이트웨이

| 구분 | DataSync | S3 파일 게이트웨이 |
| --- | --- | --- |
| 목적 | 일회성/주기적 대량 이전 | 지속적 온프레미스 ↔︎ S3 연동 |
| 방식 | 스케줄 기반, 증분 전송 | 파일 저장 시 실시간 증분 |

## 선택 기준

| 상황 | 솔루션 |
| --- | --- |
| DB 무중단 마이그레이션 | DMS (+이종이면 SCT) |
| 수십 TB, 대역폭 최소화 | Snowball Edge |
| 수백 PB, 데이터센터 전체 | Snowmobile |
| 온프레미스 파일 주기적 동기화 | DataSync |
| NFS 유지하면서 S3 확장 | S3 파일 게이트웨이 |