## Site-to-Site VPN

온프레미스 ↔︎ AWS 간 인터넷 위에 IPSec 암호화 터널.

| 장점 | 단점 |
| --- | --- |
| 빠른 구성 (수 시간) | 인터넷 경유 → 레이턴시 불안정 |
| 저렴 | 최대 1.25Gbps |

## Direct Connect

온프레미스 ↔︎ AWS 간 전용 물리 회선. 인터넷 미경유.

| 구분 | VPN | Direct Connect |
| --- | --- | --- |
| 경로 | 인터넷 경유 | 전용 물리 회선 |
| 레이턴시 | 불안정 | 일정하고 낮음 |
| 대역폭 | 최대 1.25Gbps | 1~100Gbps |
| 설치 시간 | 수 시간 | 수 주~수 개월 |

**권장 패턴:** Direct Connect + VPN 백업 → 고가용성 하이브리드 연결

## Transit Gateway

여러 VPC와 온프레미스를 중앙 허브로 연결.

```
VPC 피어링: n개 VPC = n*(n-1)/2 연결 필요 → 복잡
Transit Gateway: 모든 VPC가 TGW 하나에만 연결 → 단순
```

## 선택 기준

| 상황 | 솔루션 |
| --- | --- |
| 빠른 연결 | Site-to-Site VPN |
| 대용량, 안정성, 규정 | Direct Connect |
| 장애 대비 | Direct Connect + VPN 백업 |
| VPC 여러 개 + 온프레미스 | Transit Gateway |