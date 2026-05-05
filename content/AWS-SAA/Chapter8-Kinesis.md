# Chapter8_Kinesis

# Chapter 8. Kinesis

## SQS와 근본적 차이

| 구분 | SQS | Kinesis |
| --- | --- | --- |
| 목적 | 작업 처리 | 데이터 분석 파이프라인 |
| 처리 후 | 메시지 삭제 | 보존 기간 동안 유지 |
| 소비자 | 1개 메시지 = 1개 소비자 | 여러 소비자 동시 읽기 |

## Kinesis Data Streams

**샤드:** 처리 단위. 샤드 1개 = 초당 1MB 입력 / 2MB 출력.

**데이터 보존:** 기본 24시간, 최대 365일.

## Kinesis Data Firehose

코드 없이 S3, Redshift, OpenSearch로 자동 전달하는 완전 서버리스.

```
데이터 → Firehose → (Lambda 변환 가능) → S3/Redshift/OpenSearch
```

## 선택 기준

| 상황 | 서비스 |
| --- | --- |
| 작업 처리, 삭제 필요 | SQS |
| 실시간 분석, 여러 소비자, 재처리 | Kinesis Streams |
| 코드 없이 S3/Redshift 전달 | Kinesis Firehose |