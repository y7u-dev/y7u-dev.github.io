# 클라우드 자동화를 위한 파이썬 프로그래밍

**학습 기간:** 2026.04 ~

---

### 1. 실습 환경 준비

**주요 도구**

| 도구 | 역할 |
| --- | --- |
| Python | 자동화 스크립트 작성 언어 |
| pip | Python 패키지 설치 관리자 |
| venv | 프로젝트별 독립 가상환경 |
| AWS CLI | 터미널에서 AWS 서비스 제어 |
| boto3 | Python에서 AWS API 호출 SDK |

**가상환경 기본 명령어**

```bash
python -m venv venv          # 가상환경 생성
source venv/bin/activate     # 활성화 (Linux/Mac)
venv\Scripts\activate        # 활성화 (Windows)
deactivate                   # 비활성화

pip install boto3[crt]       # boto3 설치
pip show boto3               # 설치 확인
```

**AWS CLI 프로필 설정**

```bash
aws configure --profile lab  # 프로필 생성
aws sts get-caller-identity --profile lab  # 인증 확인
```

---

### 2. 파이썬 기초 문법

### 2-1. 변수와 자료형

**변수**
값을 저장하는 이름. 자동화 코드에서 리전, 서비스명 등을 변수로 관리

```python
region = "ap-northeast-2"
service_name = "EC2"
count = 10
is_running = True
```

**자료형 종류**

| 자료형 | 예시 | 설명 |
| --- | --- | --- |
| str | `"EC2"` | 문자열 |
| int | `10` | 정수 |
| float | `3.14` | 실수 |
| bool | `True / False` | 불린 |

**f-string 출력**

```python
instance_id = "i-0123456789"
state = "running"
print(f"ID:{instance_id}, 상태:{state}")
```

---

### 2-2. 리스트와 딕셔너리

**리스트** — 순서 있는 값의 묶음

```python
regions = ["ap-northeast-2", "ap-northeast-1", "us-east-1"]
print(regions[0])        # ap-northeast-2
print(len(regions))      # 3
regions.append("eu-west-1")
```

**딕셔너리** — key-value 쌍의 데이터 구조 (boto3 응답값이 딕셔너리 형태)

```python
instance = {"InstanceId": "i-0123", "State": "running", "Region": "ap-northeast-2"}
print(instance["InstanceId"])          # 직접 접근
print(instance.get("PublicIp", "없음")) # 없을 때 기본값 반환
```

---

### 2-3. 조건문과 반복문

**조건문**

```python
state = "running"
if state == "running":
    print("실행 중")
elif state == "stopped":
    print("중지됨")
else:
    print("기타 상태")
```

**반복문**

```python
instances = ["i-001", "i-002", "i-003"]
for instance_id in instances:
    print(instance_id)
```

**리스트 컴프리헨션** — 필터링에 자주 사용

```python
# 실행 중인 인스턴스만 추출
running = [i for i in instances if i["State"] == "running"]
```

---

### 2-4. 함수와 모듈

**함수 정의와 호출**

```python
def get_instance_info(instance):
    return {
        "id": instance.get("InstanceId"),
        "state": instance.get("State", {}).get("Name")
    }

info = get_instance_info(instance)
```

**자동화 코드에서 함수가 중요한 이유**

- 반복되는 코드를 한 곳에서 관리
- 기능별로 분리하여 수정이 쉬움
- 다른 스크립트에서 재사용 가능

**모듈 import**

```python
import json
import csv
import argparse
from botocore.exceptions import ClientError, NoCredentialsError
```

---

### 2-5. 파일 처리와 JSON 처리

**파일 모드**

| 모드 | 의미 |
| --- | --- |
| `"r"` | 읽기 |
| `"w"` | 쓰기 (덮어씀) |
| `"a"` | 추가 쓰기 |

```python
# 파일 쓰기
with open("result.txt", "w") as f:
    f.write("EC2 인벤토리 결과\n")

# 파일 읽기
with open("result.txt", "r") as f:
    content = f.read()

# JSON 저장
with open("result.json", "w") as f:
    json.dump(data, f, indent=2, ensure_ascii=False)

# JSON 읽기
with open("result.json", "r") as f:
    data = json.load(f)
```

---

### 2-6. 예외 처리

**기본 구조**

```python
try:
    response = ec2.describe_instances()
except ClientError as e:
    print(f"AWS 오류:{e.response['Error']['Code']}")
except NoCredentialsError:
    print("인증 정보 없음")
except Exception as e:
    print(f"예상치 못한 오류:{e}")
```

**자동화 코드에서 자주 보는 패턴**

```python
# 딕셔너리 키 오류 방지
value = data.get("Key", "기본값")

# 파일 없음 오류
try:
    with open("log.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    print("로그 파일이 없습니다")
```

---

### 2-7. tuple과 set

**tuple** — 변경 불가능한 리스트 (설정값, 좌표 등 고정 데이터에 사용)

```python
regions = ("ap-northeast-2", "us-east-1")
# regions[0] = "eu-west-1"  # 오류! 변경 불가

# 언패킹
name, state = ("web-server", "running")
```

**set** — 중복 없는 집합

```python
tags = {"dev", "web", "prod", "dev"}
print(tags)  # {'dev', 'web', 'prod'} 중복 제거
```

---

### 3. boto3와 AWS API 자동화 입문

**SDK / API / boto3 관계**

```
AWS 서비스 (EC2, S3, IAM 등)
    ↑ API 호출
boto3 (Python AWS SDK)
    ↑ import
Python 자동화 스크립트
```

**Session 기반 인증 (프로필 사용)**

```python
import boto3

# 프로필 기반 세션 생성 (권장)
session = boto3.Session(profile_name="lab", region_name="ap-northeast-2")
ec2 = session.client("ec2")
s3 = session.client("s3")
```

**client vs resource 방식**

| 방식 | 특징 | 사용 |
| --- | --- | --- |
| client | 저수준 API 직접 호출, 응답이 딕셔너리 | 이 과정 중심 |
| resource | 고수준 객체 방식, 직관적 | 일부 서비스 |

**AWS API 응답 구조 이해**

boto3 응답은 딕셔너리와 리스트의 중첩 구조

```python
response = ec2.describe_instances()
# response["Reservations"] → 예약 목록 (리스트)
#   ["Instances"] → 인스턴스 목록 (리스트)
#     ["InstanceId"] → 인스턴스 ID
#     ["State"]["Name"] → 상태 (running/stopped)
#     ["Tags"] → 태그 리스트
```

---

### 4. 운영 자동화 스크립트 작성 기법

**자동화 스크립트 발전 단계**

```
1. 단순 조회 코드 작성
2. main 함수 분리 (if __name__ == "__main__")
3. 클라이언트 생성 함수 분리
4. 응답 데이터 파싱 함수 분리
5. 필터링 추가
6. 예외 처리 추가
7. JSON/CSV 파일 저장
8. argparse로 명령행 인자 처리
```

**완성형 스크립트 구조**

```python
import boto3
import json
import argparse
from botocore.exceptions import ClientError, NoCredentialsError

def get_ec2_client(profile, region):
    session = boto3.Session(profile_name=profile, region_name=region)
    return session.client("ec2")

def get_instances(ec2):
    try:
        response = ec2.describe_instances()
        instances = []
        for r in response["Reservations"]:
            for i in r["Instances"]:
                instances.append({
                    "id": i.get("InstanceId"),
                    "state": i.get("State", {}).get("Name"),
                    "name": next((t["Value"] for t in i.get("Tags", [])
                                  if t["Key"] == "Name"), "없음")
                })
        return instances
    except ClientError as e:
        print(f"오류:{e.response['Error']['Code']}")
        return []

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--profile", default="lab")
    parser.add_argument("--region", default="ap-northeast-2")
    args = parser.parse_args()

    ec2 = get_ec2_client(args.profile, args.region)
    instances = get_instances(ec2)
    print(json.dumps(instances, indent=2, ensure_ascii=False))

if __name__ == "__main__":
    main()
```

**핵심 패턴 정리**

| 패턴 | 코드 | 이유 |
| --- | --- | --- |
| get() 사용 | `i.get("PublicIp", "없음")` | KeyError 방지 |
| 중첩 get | `i.get("State", {}).get("Name")` | 안전한 중첩 접근 |
| 태그 추출 | `next((t["Value"] for t in tags if t["Key"] == "Name"), "없음")` | Name 태그 안전 추출 |
| 필터링 | `[i for i in instances if i["state"] == "running"]` | 상태 기반 필터 |

---

### 5. AWS 운영/보안 점검 자동화

**IAM 주요 조회 API**

```python
iam = session.client("iam")

# 사용자 목록
response = iam.list_users()
users = response["Users"]  # PasswordLastUsed, CreateDate 포함

# 정책 목록 (로컬 정책만)
response = iam.list_policies(Scope="Local")
policies = response["Policies"]

# 사용자에 연결된 정책
response = iam.list_attached_user_policies(UserName="admin")
```

**운영 점검 자동화 패턴**

```python
# 사용자 이름 규칙 점검 (예: 소문자 + 숫자만 허용)
import re

def check_username_rule(users):
    violations = []
    for user in users:
        name = user["UserName"]
        if not re.match(r"^[a-z0-9_-]+$", name):
            violations.append(name)
    return violations
```

---

### 6. 로그 파일 분석 자동화

**로그 유형**

| 유형 | 형식 | 예시 |
| --- | --- | --- |
| 텍스트 로그 | 한 줄씩 텍스트 | 시스템/앱 로그 |
| JSON 로그 | JSON 구조 | CloudTrail |
| 액세스 로그 | IP 경로 상태코드 | ALB, Nginx |
| 에러 로그 | 오류 레벨 메시지 | 애플리케이션 오류 |

**로그 분석 기본 패턴**

```python
# ERROR 로그만 출력
with open("app.log", "r") as f:
    for line in f:
        if "ERROR" in line:
            print(line.strip())

# 로그 레벨별 처리
for line in lines:
    if "ERROR" in line:
        print(f"[오류]{line}")
    elif "WARNING" in line:
        print(f"[경고]{line}")
    elif "INFO" in line:
        print(f"[정보]{line}")
```

**정규표현식 활용**

```python
import re

# 자주 사용하는 패턴
# \d+     숫자 하나 이상
# \w+     영문자/숫자/언더스코어
# .+      임의 문자 하나 이상
# ^       줄 시작
# $       줄 끝

# IP 추출 예시
pattern = r"\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"
ip = re.findall(pattern, log_line)
```

---

### 7. AWS 로그 분석과 생성형 AI 활용

**CloudWatch Logs 조회 흐름**

```
boto3 CloudWatch Logs client 생성
    ↓
describe_log_streams() → 로그 스트림 목록 조회
    ↓
최신 스트림 이름 추출
    ↓
get_log_events(logStreamName=...) → 이벤트 조회
    ↓
메시지 추출 및 분석
```

**주요 API**

```python
logs = session.client("logs")

# 로그 스트림 목록
streams = logs.describe_log_streams(
    logGroupName="/aws/my-app",
    orderBy="LastEventTime",
    descending=True
)["logStreams"]

# 최신 스트림 이름
latest_stream = streams[0]["logStreamName"]

# 로그 이벤트 조회
events = logs.get_log_events(
    logGroupName="/aws/my-app",
    logStreamName=latest_stream,
    limit=100
)["events"]

messages = [e["message"] for e in events]
```

**ALB Access Log 특징**
- S3에 자동 저장
- 공백 구분 포맷 (15개 필드)
- boto3로 S3에서 읽어 파싱

---

### 8. Amazon Bedrock 기본 이해

**Amazon Bedrock이란?**
AWS에서 여러 Foundation Model을 API로 호출할 수 있게 해주는 완전관리형 생성형 AI 서비스
모델 서버를 직접 운영하지 않고 API만으로 사용

**client 종류**

| client | 역할 |
| --- | --- |
| `bedrock` | 모델 목록 조회, 모델 정보 확인 |
| `bedrock-runtime` | 실제 모델 호출 (텍스트 생성) |

**Converse API 기본 구조**

```python
bedrock_runtime = session.client("bedrock-runtime")

response = bedrock_runtime.converse(
    modelId="anthropic.claude-3-5-sonnet-20241022-v2:0",
    messages=[
        {
            "role": "user",
            "content": [{"text": "EC2 인스턴스 상태를 요약해줘"}]
        }
    ],
    inferenceConfig={
        "maxTokens": 1000,
        "temperature": 0.1
    }
)

# 응답 텍스트 추출
result = response["output"]["message"]["content"][0]["text"]
```

**운영 자동화 활용 흐름**

```
운영 데이터 수집 (boto3로 EC2/CloudWatch 조회)
    ↓
프롬프트 구성 (데이터 + 질문)
    ↓
Bedrock Runtime 호출 (Converse API)
    ↓
자연어 요약 결과 생성
    ↓
파일 저장 또는 Slack 전송
```

---

### 9. Amazon Bedrock과 RAG를 이용한 운영 챗봇

**RAG(Retrieval-Augmented Generation)가 필요한 이유**

LLM은 학습 데이터 기반으로만 답변 → 우리 회사 내부 운영 문서 모름
RAG = 질문 → 관련 문서 검색 → 검색 결과 + 질문을 LLM에 전달 → 답변

```
일반 방식:  질문 → LLM → 답변 (내부 문서 모름)
RAG 방식:   질문 → 문서 검색 → 검색 결과 + 질문 → LLM → 정확한 답변
```

**운영 챗봇 질문 유형**

| 유형 | 예시 | 처리 방법 |
| --- | --- | --- |
| 문서형 | “배포 절차가 뭐야?” | Knowledge Base RAG |
| 상태형 | “지금 EC2 몇 대야?” | Python boto3 조회 |
| 혼합형 | “현재 오류 서버 대응 방법은?” | 상태 조회 + RAG + LLM |

**Knowledge Base 구성 흐름**

```
운영 문서 작성 (.txt, .pdf 등)
    ↓
S3 버킷에 업로드
    ↓
Bedrock Knowledge Base 생성 (S3 연결)
    ↓
임베딩 모델로 벡터화 → 벡터 DB 저장
    ↓
retrieve_and_generate() 로 질의응답
```

**RetrieveAndGenerate API**

```python
kb_client = session.client("bedrock-agent-runtime")

response = kb_client.retrieve_and_generate(
    input={"text": "배포 롤백 절차가 어떻게 돼?"},
    retrieveAndGenerateConfiguration={
        "type": "KNOWLEDGE_BASE",
        "knowledgeBaseConfiguration": {
            "knowledgeBaseId": "your-kb-id",
            "modelArn": "arn:aws:bedrock:ap-northeast-2::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0"
        }
    }
)

answer = response["output"]["text"]
```

**하이브리드 챗봇 구조**

```
사용자 질문
    ↓
질문 유형 분류
    ├── 문서형 → Knowledge Base RAG
    ├── 상태형 → boto3 조회 함수
    └── 혼합형 → 상태 조회 + RAG + Bedrock 응답 생성
```

---

### 10. 핵심 정리

| 개념 | 한 줄 요약 |
| --- | --- |
| boto3 | Python에서 AWS API 호출하는 공식 SDK |
| Session | 프로필 기반 인증 세션. 여러 client 생성에 재사용 |
| client 방식 | 저수준 API 직접 호출, 응답은 딕셔너리 |
| get() | KeyError 방지. 없으면 기본값 반환 |
| argparse | 명령행 인자 처리. 스크립트 재사용성 향상 |
| if **name** == “**main**” | 직접 실행 시에만 main() 호출 |
| CloudWatch Logs | boto3로 로그 스트림 조회 및 이벤트 추출 |
| Bedrock | 생성형 AI 모델 API 서비스. 운영 데이터 요약에 활용 |
| Converse API | Bedrock 모델 호출 방식. 메시지 기반 일관된 구조 |
| RAG | 외부 문서 검색 결과를 LLM 답변에 활용 |
| Knowledge Base | S3 문서를 벡터화하여 RAG 검색 소스로 활용 |