# Chapter3_스토리지

# Chapter 3. 스토리지

## 세 가지 스토리지의 근본적 차이

| 구분 | EBS | EFS | S3 |
| --- | --- | --- | --- |
| 종류 | 블록 스토리지 | 파일 스토리지 | 객체 스토리지 |
| 접근 방식 | OS가 디스크로 인식 | NFS 마운트 | HTTP API |
| 공유 | 단일 EC2 | 여러 EC2 동시 | 제한 없음 |
| 용량 | 미리 지정 | 자동 확장 | 무제한 |

## EFS 스토리지 클래스

| 클래스 | 특징 |
| --- | --- |
| EFS Standard | 자주 접근, 빠른 레이턴시 |
| EFS Infrequent Access (IA) | 자동 이동, 비용 92% 절감 |

## S3 스토리지 클래스

| 클래스 | 복원 시간 | 용도 |
| --- | --- | --- |
| S3 Standard | 즉시 | 자주 접근 |
| S3 Standard-IA | 즉시 | 한 달에 한 번 접근 |
| S3 One Zone-IA | 즉시 | 재생성 가능 데이터 |
| S3 Glacier Instant Retrieval | 즉시 | 분기에 한 번 접근 |
| S3 Glacier Flexible Retrieval | 수 분~수 시간 | 연 1~2회 접근 |
| S3 Glacier Deep Archive | 12시간+ | 7년 이상 법적 보관 |

## S3 수명 주기 정책

```
생성 후 30일  → Standard-IA
생성 후 90일  → Glacier
생성 후 365일 → 삭제
```

## EBS 스냅샷

- S3에 저장 → 다른 AZ/리전 복사 가능
- 증분 방식 (변경된 부분만 저장)
- 재해 복구 설계의 핵심