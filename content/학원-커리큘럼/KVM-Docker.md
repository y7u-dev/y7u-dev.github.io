# KVM 및 Docker 가상화

---

**학습 기간:** 2026.02.10 ~ 2026.02.22

---

### 1. 가상화 (Virtualization)

**정의**
하나의 물리적 하드웨어 자원을 논리적으로 분할하여 여러 개의 독립적인 실행 환경(VM)을 동시에 운영하는 기술.

**등장 배경**

| 문제점 | 설명 |
| --- | --- |
| 낮은 자원 활용률 | 서버 1대에 서비스 1개 → CPU 사용률 5~10% |
| 서버 증설 비용 | 서비스 증가 = 서버 구매 |
| 운영 복잡도 | 서버 수 증가 → 관리 포인트 증가 |

**하이퍼바이저 유형**

| 구분 | Type 1 (Bare-metal) | Type 2 (Hosted) |
| --- | --- | --- |
| 설치 위치 | 하드웨어 위 | Host OS 위 |
| 성능 | 높음 | 상대적으로 낮음 |
| 사용 환경 | 서버, 클라우드 | 개인 PC, 실습 |
| 예시 | KVM, Xen, ESXi | VirtualBox, VMware Workstation |

**VM 구성 요소**

| 요소 | 설명 |
| --- | --- |
| vCPU | 물리 CPU를 논리적으로 분할 |
| vMemory | 실제 RAM 일부 할당 |
| vDisk | 파일 형태의 디스크 |
| vNIC | 가상 네트워크 인터페이스 |

---

### 2. KVM

**정의**
Linux 커널에 내장된 Type 1 하이퍼바이저. Intel VT-x / AMD-V CPU 가상화 기능을 활용.

**KVM의 가상화 방식**

- 기본 방식: **전가상화** (Guest OS 수정 불필요, 기존 OS 그대로 실행)
- I/O 최적화: **Virtio 드라이버** (반가상화 드라이버) 병행 사용
    - virtio-net (네트워크), virtio-blk (디스크), virtio-balloon (메모리)

| 항목 | KVM |
| --- | --- |
| 가상화 방식 | 전가상화 |
| Guest OS 수정 | 불필요 |
| I/O 최적화 | Virtio (반가상화 드라이버) |

---

### 3. 컨테이너 (Container)

**VM vs 컨테이너**

| 구분 | VM | 컨테이너 |
| --- | --- | --- |
| 가상화 대상 | 하드웨어 | OS 커널 |
| OS 포함 여부 | 포함 | 미포함 |
| 부팅 속도 | 수십 초 ~ 분 | 수 초 |
| 리소스 사용 | 무거움 | 매우 가벼움 |

**컨테이너 핵심 기술**

**① Namespace** — 격리
리눅스 시스템을 여러 독립 공간으로 나누는 기능. 컨테이너마다 다른 namespace를 가져 프로세스·네트워크·파일시스템이 분리된 것처럼 동작.

| Namespace | 역할 |
| --- | --- |
| PID | 프로세스 ID 격리 |
| NET | 네트워크 인터페이스 격리 |
| MNT | 파일시스템 마운트 격리 |
| UTS | hostname 격리 |

**② cgroups** — 자원 제한
CPU, 메모리, 디스크 I/O 사용량을 커널이 강제로 제한.

> **컨테이너 = Namespace(격리) + cgroups(제어)**
> 

**컨테이너 이미지 레이어 구조**

```
[ App Layer        ]
[ Runtime Layer    ]
[ OS Base Layer    ]  ← 읽기 전용
─────────────────────
[ Container RW Layer ] ← 컨테이너 실행 시 추가
```

같은 이미지를 여러 컨테이너가 공유 가능 → 디스크 절약

---

### 4. Docker

**정의**
애플리케이션과 실행 환경을 이미지로 패키징하여 어디서든 동일하게 실행할 수 있게 해주는 컨테이너 플랫폼.

> "내 PC에서는 되는데요?" 문제를 해결
> 

**핵심 명령어**

```bash
# 이미지 관련
docker pull nginx              # 이미지 다운로드
docker images                  # 이미지 목록
docker rmi <이미지ID>           # 이미지 삭제

# 컨테이너 생명주기
docker run -d --name web01 -p 8080:80 nginx   # 실행
docker ps                      # 실행 중 컨테이너
docker ps -a                   # 전체 컨테이너
docker stop web01              # 중지
docker start web01             # 시작
docker rm web01                # 삭제
docker rm -f web01             # 강제 삭제 (stop + rm)

# 컨테이너 접속
docker exec -it web01 bash     # 내부 접속 (권장)

# 로그/상태
docker logs -f web01           # 실시간 로그
docker inspect web01           # 상세 정보
docker stats                   # 리소스 사용량
```

**docker run 주요 옵션**

| 옵션 | 의미 |
| --- | --- |
| `-d` | 백그라운드 실행 |
| `-it` | 터미널 연결 (쉘 접속용) |
| `--name` | 컨테이너 이름 지정 |
| `-p 8080:80` | 포트 포워딩 (호스트:컨테이너) |
| `--rm` | 종료 시 자동 삭제 |

> `docker run = pull + create + start + attach`
> 

---

### 5. Dockerfile

이미지를 만들기 위한 설계서. 위에서 아래로 순차 실행, 한 줄 = 한 레이어.

```docker
FROM ubuntu:22.04          # 베이스 이미지 (필수)
WORKDIR /app               # 작업 디렉터리
RUN apt update && apt install -y nginx   # 빌드 시 실행
COPY index.html /usr/share/nginx/html/  # 파일 복사
ENV APP_ENV=production     # 환경 변수 (런타임 유지)
ARG VERSION                # 빌드 타임 변수
EXPOSE 80                  # 포트 문서화
CMD ["nginx", "-g", "daemon off;"]      # 기본 실행 명령
```

```bash
docker build -t myimage:1.0 .   # 이미지 빌드
```

---

### 6. 컨테이너 데이터 관리

컨테이너 삭제 시 내부 데이터도 함께 삭제됨 → **데이터는 반드시 컨테이너 외부로 분리**

| 방식 | 용도 | 관리 주체 | 이식성 |
| --- | --- | --- | --- |
| Volume | 운영 표준 | Docker | 높음 |
| Bind Mount | 개발/테스트 | 사용자 | 낮음 |
| docker cp | 임시/응급 | - | - |

```bash
# Volume 생성 및 연결
docker volume create myvol
docker run -d --name web \
  --mount type=volume,source=myvol,target=/usr/share/nginx/html \
  nginx

# Bind Mount
docker run -d --name web \
  --mount type=bind,source=/home/user/web,target=/usr/share/nginx/html \
  nginx
```

> Volume 동작 규칙: "비어 있으면 채우고, 차 있으면 그대로 쓴다."
> 

---

### 7. Docker 네트워크

| 드라이버 | 설명 | 사용 목적 |
| --- | --- | --- |
| bridge | 가상 브리지 기반 (기본값) | 일반 컨테이너 통신 |
| host | 호스트 네트워크 공유 | 성능 최우선 |
| none | 네트워크 없음 | 완전 격리 |

```bash
# 컨테이너 통신 흐름
Container → veth → bridge(docker0) → host NIC

# 포트 포워딩 (외부 접근용)
-p 8080:80   # 호스트 8080 → 컨테이너 80
```

**사용자 정의 네트워크** — 컨테이너 이름으로 DNS 통신 가능 (권장)

```bash
docker network create mynet
docker run --network mynet --name web nginx
docker run --network mynet --name db mysql
# web 컨테이너에서 db 이름으로 접근 가능
```

---

### 8. Docker Compose

여러 컨테이너를 하나의 서비스 스택으로 정의하고 관리하는 도구.

> "docker run을 코드로 작성한 것이 Docker Compose다."
> 

```yaml
# docker-compose.yml 기본 구조
services:
  web:
    image: nginx:latest
    container_name: web01
    ports:
      - "8080:80"
    volumes:
      - webdata:/usr/share/nginx/html
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=1234
    volumes:
      - dbdata:/var/lib/mysql

volumes:
  webdata:
  dbdata:
```

```bash
docker compose up -d      # 전체 서비스 실행
docker compose down       # 전체 서비스 중지 및 삭제
docker compose ps         # 서비스 상태 확인
docker compose logs -f    # 전체 로그
```

> Compose는 서비스 이름을 DNS로 자동 등록 → `web`에서 `db`로 이름으로 직접 접근 가능
> 

---