# Chapter13_IAM

# Chapter 13. IAM

## 구성 요소

| 요소 | 역할 |
| --- | --- |
| User | 실제 사람/앱, 아이디+패스워드 or Access Key |
| Group | 사용자 묶음, 그룹에 정책 연결 |
| Role | 서비스/외부에 임시 권한 부여, Access Key 불필요 |
| Policy | 권한 정의 JSON 문서 |

## 권한 평가 원리

```
기본: 모든 것이 거부 (Deny)
1. 명시적 Deny → 무조건 거부 (최우선)
2. 명시적 Allow → 허용
3. 아무것도 없음 → 묵시적 Deny
```

## 보안 모범 사례

1. 루트 계정 최소화 (MFA 설정 후 잠금)
2. **최소 권한 원칙** (꼭 필요한 권한만)
3. Access Key 대신 Role 사용
4. MFA 활성화
5. 권한 경계(Permission Boundary)로 최대 권한 제한

## AWS Organizations

**OU (Organizational Unit):** 계정을 묶는 폴더 단위

**SCP (Service Control Policy):**
- 조직/OU에 최대 권한 제한
- IAM 정책보다 상위
- SCP 거부 → IAM Allow 있어도 차단

## S3 버킷 정책 조건 키

| 조건 키 | 범위 |
| --- | --- |
| aws:PrincipalOrgID | 조직 전체 |
| aws:PrincipalOrgPaths | 특정 OU |
| aws:PrincipalTag | 태그 기준 |

## OIDC

외부 서비스(GitHub Actions 등)가 Access Key 없이 신원 기반으로 임시 권한을 받아 AWS에 접근하는 표준 방식.