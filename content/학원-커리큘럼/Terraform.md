# 배포자동화를_위한_Terraform

**학습 기간:** 2026.04.21 ~

---

### 1. IaC와 Terraform 개요

**IaC(Infrastructure as Code)란?**
인프라 구성을 콘솔에서 직접 클릭하지 않고, 코드로 정의하고 코드 기준으로 관리하는 방식

**수작업 방식의 한계**

- 같은 구성을 dev / stg / prod 환경에 반복 적용하기 어려움
- 누가 언제 어떤 설정으로 만들었는지 추적 불가
- 환경마다 설정이 조금씩 달라질 수 있음 (일관성 부족)
- 동일한 구조를 다시 만들어야 할 때 재현 어려움

**IaC의 핵심 장점**

| 특성 | 설명 |
| --- | --- |
| 선언형 관리 | “어떻게 만들 것인가”가 아니라 “어떤 상태가 되어야 하는가”를 정의 |
| 재현성 | 같은 코드로 동일한 인프라를 반복 생성 가능 |
| 버전 관리 | Git으로 변경 이력 추적, 코드 리뷰, 롤백 가능 |
| 표준화 | 이름 규칙, 태그 정책, 보안 정책을 코드 수준에서 반복 적용 |
| 자동화 | CI/CD 파이프라인과 결합하여 인프라 배포 자동화 |

**Terraform이란?**
HashiCorp가 만든 범용 IaC 도구. AWS뿐 아니라 GCP, Azure, Kubernetes, GitHub, Datadog 등 다양한 플랫폼을 **HCL(HashiCorp Configuration Language)** 이라는 단일 언어로 관리

**Terraform vs CloudFormation**

| 항목 | Terraform | CloudFormation |
| --- | --- | --- |
| 개발 주체 | HashiCorp | AWS |
| 지원 범위 | 멀티 클라우드 및 다양한 플랫폼 | AWS 중심 |
| 구성 언어 | HCL | YAML / JSON |
| 상태 관리 | 별도 state 파일 | AWS 스택 상태 기반 |
| 확장성 | 높음 | AWS 내부 운영에 최적화 |

---

### 2. Terraform 핵심 구성 요소

**Provider**
Terraform이 어떤 플랫폼과 통신할지 정의. AWS를 사용하려면 AWS Provider 필요

```hcl
provider "aws" {
  region = "ap-northeast-2"
}
```

**Resource**
실제로 생성하거나 관리할 인프라 자원 (EC2, VPC, SG, IAM 등)

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0123456789abcdef0"
  instance_type = "t3.micro"
  tags = { Name = "tf-web-01" }
}
```

**Data Source**
이미 존재하는 정보를 조회할 때 사용 (Resource는 만든다, Data Source는 찾는다)

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}
```

**Variable / Output / Locals**

| 요소 | 역할 | 사용 예 |
| --- | --- | --- |
| variable | 외부에서 주입받는 입력값 | 인스턴스 타입, CIDR 블록 |
| output | 실행 결과에서 중요한 값 출력 | EC2 퍼블릭 IP, VPC ID |
| locals | 코드 내부에서만 쓰는 계산값 | 이름 접두사 조합 |

```hcl
variable "instance_type" { default = "t3.micro" }
output "instance_id" { value = aws_instance.web.id }
locals { name_prefix = "dev-${var.env}" }
```

**변수 값 전달 우선순위**
CLI `-var` > `terraform.tfvars` > 환경 변수 > `default`

**State**
Terraform이 관리하는 인프라 상태를 기록하는 파일 (`terraform.tfstate`)
코드 + state + 실제 인프라 3가지를 비교하여 변경 내용을 계산

---

### 3. Terraform 기본 명령어

```
terraform init     # Provider 플러그인 다운로드, 작업 디렉터리 초기화
terraform plan     # 변경 예정 사항 계산 및 출력 (실제 반영 X)
terraform apply    # plan 결과를 실제 인프라에 반영
terraform destroy  # Terraform이 만든 자원 삭제
```

**plan 결과에서 확인할 것**

- `+` 새로 생성될 자원
- `~` 변경될 자원
- 삭제될 자원

**주요 옵션**

```
terraform apply -auto-approve    # 확인 없이 바로 반영
terraform plan -out=tfplan       # plan 결과 파일로 저장
terraform apply tfplan           # 저장된 plan 그대로 적용
```

---

### 4. 파일 구조와 블록

**기본 파일 분리 구조**

```
main.tf          # 주요 리소스 정의
variables.tf     # 변수 선언
outputs.tf       # 출력값 선언
terraform.tfvars # 변수 실제 값
provider.tf      # Provider 설정
```

Terraform은 같은 디렉터리 내 모든 `.tf` 파일을 하나처럼 읽음

**블록 기본 구조**

```hcl
<블록 타입> "<블록 레이블>" "<로컬 이름>" {
  인자 = 값
}
```

**required_providers 버전 표현**

| 표현 | 의미 |
| --- | --- |
| `>= 1.5.0` | 1.5.0 이상 |
| `~> 5.0` | 5.x 허용, 6.0 미만 |
| `>= 1.0, < 2.0` | 범위 지정 |

---

### 5. AWS 인프라 구성 흐름

**VPC → Subnet → IGW → Route Table → EC2 순서**

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

# Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-northeast-2a"
  map_public_ip_on_launch = true
}

# Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

# Route Table 연결
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

**리소스 간 참조**`aws_vpc.main.id` 처럼 `리소스타입.로컬이름.속성` 형태로 다른 리소스 값을 참조

---

### 6. 반복 구성 (count / for_each)

**count**
숫자로 반복. 인덱스로 구분

```hcl
resource "aws_subnet" "public" {
  count      = 2
  cidr_block = "10.0.${count.index}.0/24"
}
```

**for_each**
map 또는 set으로 반복. key/value로 구분

```hcl
resource "aws_subnet" "public" {
  for_each          = toset(["ap-northeast-2a", "ap-northeast-2b"])
  availability_zone = each.key
}
```

**count vs for_each**

| 항목 | count | for_each |
| --- | --- | --- |
| 식별 방식 | 인덱스 번호 | 고유 key |
| 중간 삭제 시 | 이후 인덱스 전체 변경 위험 | 해당 key만 삭제 |
| 권장 상황 | 단순 반복 | 식별 가능한 리소스 관리 |

**조건식**

```hcl
# 삼항 연산자
instance_type = var.env == "prod" ? "t3.medium" : "t3.micro"

# 조건부 리소스 생성
count = var.create_nat ? 1 : 0
```

---

### 7. 모듈화

**모듈이란?**
반복되는 Terraform 코드를 재사용 가능한 단위로 묶은 것. 모든 Terraform 디렉터리는 모듈

- root module: 실행 진입점 (terraform apply를 실행하는 디렉터리)
- child module: root에서 호출되는 재사용 모듈

**모듈 디렉터리 구조**

```
modules/
  vpc/
    main.tf       # 리소스 정의
    variables.tf  # 입력값 선언
    outputs.tf    # 출력값 선언
main.tf           # 모듈 호출
```

**모듈 호출**

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
  env    = var.env
}

# 모듈 출력값 참조
output "vpc_id" {
  value = module.vpc.vpc_id
}
```

**모듈 source 옵션**

| 유형 | 예시 |
| --- | --- |
| 로컬 경로 | `./modules/vpc` |
| Terraform Registry | `terraform-aws-modules/vpc/aws` |
| GitHub | `github.com/org/repo` |

---

### 8. State와 Backend

**State의 역할**

- Terraform이 어떤 자원을 만들었는지 기록
- 코드와 실제 인프라 상태를 비교하는 기준
- output 값 관리

**local state의 한계**

- 내 PC에만 있어서 팀 협업 불가
- 동시 실행 시 충돌 위험
- PC 분실 시 유실 위험

**remote backend (S3 + DynamoDB)**

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tfstate-bucket"
    key            = "prod/terraform.tfstate"
    region         = "ap-northeast-2"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```

- S3: state 파일 저장
- DynamoDB: 동시 실행 방지를 위한 Lock 관리

**drift**
Git 코드 상태와 실제 클라우드 상태가 달라진 것
`terraform plan`으로 감지, `terraform apply`로 복구

**state 관련 명령어**

```
terraform state list          # 관리 중인 리소스 목록
terraform state show <리소스>  # 특정 리소스 상태 확인
terraform state rm <리소스>    # state에서 리소스 제거 (실제 삭제 X)
terraform import <리소스> <ID> # 기존 리소스를 state로 가져오기
```

**workspace**
동일한 코드로 dev / stg / prod 환경을 분리 관리할 때 사용

```
terraform workspace new dev     # 새 workspace 생성
terraform workspace select dev  # workspace 전환
terraform workspace list        # 목록 확인
```

---

### 9. Ansible 기본과 Terraform 연계

**Ansible이란?**
에이전트 없이 SSH로 서버에 접속하여 소프트웨어 설치, 설정, 서비스 관리를 자동화하는 구성 관리 도구

**Terraform vs Ansible 역할 분리**

| 도구 | 역할 | 담당 영역 |
| --- | --- | --- |
| Terraform | 인프라 프로비저닝 | EC2, VPC, SG, RDS 생성 |
| Ansible | 구성 관리 | OS 설정, 패키지 설치, 앱 배포 |

**Ansible 핵심 개념**

- Inventory: 관리할 서버 목록 (IP 또는 도메인)
- Playbook: 수행할 작업을 YAML로 정의한 파일
- Task: Playbook 안의 개별 실행 단위
- Module: 파일 복사, 패키지 설치 등 기능 단위

**Terraform + Ansible 연계 흐름**

```
1. Terraform으로 EC2 생성
2. Terraform output으로 EC2 IP 출력
3. Ansible Inventory에 IP 등록
4. Ansible Playbook으로 Nginx 설치 및 설정 자동화
```

**간단한 Playbook 예시**

```yaml
-hosts: web
become:true
tasks:
-name: Install Nginx
apt:
name: nginx
state: present

-name: Start Nginx
service:
name: nginx
state: started
enabled:true
```

---

### 10. 핵심 정리

| 개념 | 한 줄 요약 |
| --- | --- |
| IaC | 인프라를 코드로 선언하고 관리 |
| Provider | Terraform이 통신할 플랫폼 정의 |
| Resource | 실제 생성/관리할 자원 |
| Data Source | 이미 존재하는 정보 조회 |
| State | Terraform이 관리하는 인프라 상태 기록 |
| Backend | State를 원격(S3 등)에 저장하는 설정 |
| Module | 재사용 가능한 코드 묶음 |
| for_each | key 기반 반복 (count보다 안전) |
| drift | 코드와 실제 인프라 상태 불일치 |
| Ansible | Terraform이 만든 서버에 소프트웨어 설치/설정 |