## Lambda

이벤트 발생 시 코드 실행, 사용한 만큼만 비용 청구.

**트리거:** API Gateway, S3, SQS, SNS, EventBridge, DynamoDB, Kinesis

**제한:**
- 최대 실행 시간: 15분
- 메모리: 128MB ~ 10GB

### 콜드 스타트

오래 안 쓰다가 실행 시 컨테이너 생성으로 수십 ms ~ 수 초 지연.

**해결 방법: Provisioned Concurrency**
- 컨테이너를 미리 띄워두고 항상 웜 상태 유지
- 예약 조정과 조합: 업무 시작 전 늘림 / 업무 후 줄임 → 비용 최소화

## API Gateway

Lambda를 HTTP 엔드포인트로 노출. 인증/인가, 속도 제한, 캐싱 제공.

| 타입 | 특징 | 용도 |
| --- | --- | --- |
| REST API | 기능 풍부, 캐싱 | 일반 웹/모바일 백엔드 |
| HTTP API | 단순, 70% 저렴 | 기본 기능만 필요 |
| WebSocket API | 양방향 실시간 | 채팅, 실시간 알림 |

## EC2 vs Lambda 선택 기준

| EC2 적합 | Lambda 적합 |
| --- | --- |
| 실행 시간 15분 이상 | 이벤트 기반 간헐적 실행 |
| 특정 OS/런타임 필요 | 트래픽 변동 극심 |
| GPU 필요 | 운영 오버헤드 최소화 |

## 완전 서버리스 아키텍처

```
[클라이언트] → [Route 53] → [CloudFront] → [API Gateway]
                                                  ↓
                                             [Lambda]
                                           ↙         ↘
                                    [DynamoDB]      [S3]
```