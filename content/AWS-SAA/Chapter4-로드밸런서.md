## 로드밸런서가 해결하는 문제

- 트래픽 분산 (과부하 방지)
- 단일 장애점 제거
- 무중단 배포
- SSL 종료 (HTTPS 복호화를 ALB가 대신)
- 헬스 체크 (문제 서버 자동 제외)

## ALB vs NLB vs GLB

| 구분 | ALB | NLB | GLB |
| --- | --- | --- | --- |
| 계층 | 7계층 (HTTP) | 4계층 (TCP/UDP) | 3계층 |
| 라우팅 기준 | URL 경로, 호스트, 헤더 | IP/포트 | - |
| 고정 IP | 없음 | 있음 | - |
| 용도 | 웹앱, 마이크로서비스 | 게임, IoT, 금융 | 보안 장비 통과 |

## ALB 경로 기반 라우팅

```
ALB
├── /api/*    → 백엔드 EC2 그룹
├── /static/* → S3
└── /         → 프론트엔드 EC2 그룹
```

## SSL 종료 (SSL Termination)

```
클라이언트 → HTTPS → ALB → HTTP → EC2
```

인증서 관리는 **ACM(AWS Certificate Manager)** 으로 자동 발급/갱신.

> **CloudFront에 ACM 인증서 사용 시 반드시 us-east-1 리전에서 발급해야 함**
>