# Chapter15_모니터링_운영

# Chapter 15. 모니터링 / 운영

## CloudWatch

| 기능 | 설명 |
| --- | --- |
| 메트릭 | CPU, 네트워크 등 수치 데이터 자동 수집 |
| 알람 | 임계값 초과 시 SNS 알림 or Auto Scaling 트리거 |
| 로그 | 앱/서비스 로그 중앙 수집, Logs Insights로 분석 |
| 대시보드 | 여러 메트릭 시각화 |

> 메모리, 디스크 사용률은 기본 제공 안 됨 → CloudWatch Agent 설치 필요
> 

## CloudTrail

AWS API 호출 기록. **누가 + 언제 + 어디서 + 무엇을 + 결과**

**CloudWatch vs CloudTrail:**
- CloudWatch: “지금 CPU가 몇 %인가?” (상태 모니터링)
- CloudTrail: “누가 버킷을 삭제했나?” (행동 감사)

장기 보관: S3에 저장 + KMS 암호화 (로그 위변조 방지)

## AWS Config

리소스 설정이 규정을 준수하는지 지속 감시.

```
"S3 버킷 퍼블릭 접근 차단 켜져 있나?"
위반 감지 → 알림 or Auto Remediation (자동 수정)
```

**CloudTrail vs AWS Config:**
- CloudTrail: “누가 뭘 했나” (행동 추적)
- AWS Config: “지금 설정이 올바른가” (설정 상태 추적)

## 자동화 패턴

**자동 장애 대응:**

```
CloudWatch (CPU > 90%) → 알람 → SNS 알림 + Auto Scaling
```

**보안 이상 감지:**

```
CloudTrail (루트 계정 로그인) → EventBridge → Lambda → SNS 알림
```

**규정 준수 자동화:**

```
AWS Config (S3 퍼블릭 접근 감지) → Auto Remediation → 자동 차단
```