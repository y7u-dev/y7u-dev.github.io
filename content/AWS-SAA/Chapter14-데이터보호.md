# Chapter14_데이터_보호

# Chapter 14. 데이터 보호

## 암호화의 두 가지 상태

| 상태 | 의미 | 적용 |
| --- | --- | --- |
| At Rest | 저장 중 암호화 | EBS, S3, RDS |
| In Transit | 전송 중 암호화 | HTTPS, TLS |

## KMS (Key Management Service)

암호화 키 관리. KMS 키는 KMS 밖으로 절대 나가지 않음.

| 키 종류 | 특징 |
| --- | --- |
| AWS 관리형 키 | 자동 생성/관리, 무료 |
| 고객 관리형 키(CMK) | 직접 생성, 세밀한 접근 제어, 비용 발생 |
| 가져온 키 | 온프레미스 생성 키 사용 |

## Secrets Manager

자격 증명(DB 비밀번호, API 키) 저장 + 자동 로테이션.

**KMS와 차이:**
- KMS: 암호화 키 관리
- Secrets Manager: 자격 증명 저장

**자동 로테이션:** 주기적으로 새 비밀번호 생성 → DB 적용 → 앱은 항상 최신 값 가져옴

> **“자동 회전” 키워드 → Secrets Manager**
Parameter Store는 저장 가능하지만 자동 회전 없음
> 

## ACM (AWS Certificate Manager)

SSL/TLS 인증서 자동 발급/갱신. ALB, CloudFront에만 사용 가능.

> **CloudFront + ACM: 반드시 us-east-1 리전에서 발급**
>