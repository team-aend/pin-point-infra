# 📌 pin-point-infra

위치 기반 신규 매장 실시간 푸시 알림 서비스 **`pin-point`** 프로젝트의 로컬 개발 인프라 환경을 구축하기 위한 저장소입니다. Docker Compose를 사용하여 백엔드(`backend`) 및 크롤러(`crawling`) 애플리케이션이 공통으로 사용하는 데이터베이스(MySQL)와 인메모리 캐시(Redis) 인프라를 컨테이너 환경으로 일관되게 제공합니다.

## 🛠️ 사전 준비 사항 (Prerequisites)

이 인프라를 실행하기 전에 로컬 컴퓨터에 아래 프로그램이 설치되어 있어야 합니다.

- **Docker Desktop** (기본적으로 **Docker Compose V2**가 포함되어 함께 설치됩니다.)
  - [Docker Desktop 공식 다운로드 주소](https://www.docker.com/products/docker-desktop/)
  - *참고 (Intel Mac / macOS Ventura 13 환경):* 최신 버전 설치 시 호환성 에러가 발생할 수 있으므로, [Docker Release Notes Archive](https://docs.docker.com/desktop/release-notes/)에서 **v4.30.0 이하 Intel chip 버전**을 직접 다운로드하여 설치하는 것을 권장합니다.

## 👨‍💻 팀원 로컬 개발 워크플로우 (How to Work)

API(백엔드) 및 크롤러 개발자는 코드를 실행하기 위해 반드시 이 인프라 환경을 먼저 로컬에 띄워두어야 합니다. 아래 순서대로 로컬 개발 환경을 세팅해 주세요.

### 1단계: 레포지토리 클론
작업할 상위 폴더(workspace)에 인프라 레포지토리와 본인이 담당할 애플리케이션 레포지토리를 모두 클론합니다.
```bash
# 1. 인프라 기지 클론
git clone [https://github.com/본인유저명/pin-point-infra.git](https://github.com/본인유저명/pin-point-infra.git)

# 2. 본인 담당 프로젝트 클론 (예: 백엔드)
git clone [https://github.com/본인유저명/pin-point-backend.git](https://github.com/본인유저명/pin-point-backend.git)

```

### 2단계: 인프라 기지 가동 (출근 후 1회 실행)

터미널에서 `pin-point-infra` 폴더로 이동하여 도커 컨테이너를 백그라운드에서 실행합니다.

```bash
cd pin-point-infra
docker-compose up -d

```

> 💡 **확인 방법:** `docker ps`를 입력하여 `pinpoint-mysql`과 `pinpoint-redis`가 `Up` 상태인지 확인합니다.

### 3단계: 애플리케이션 설정 및 개발 시작

본인이 담당하는 애플리케이션 레포지토리(예: `pin-point-backend`)를 IDE(VS Code, IntelliJ 등)로 엽니다. 로컬에 띄워둔 도커 인프라의 포트(MySQL: `3307`, Redis: `6379`)에 맞추어 연결 설정을 완료한 후, 비즈니스 로직 코딩 및 서버 구동을 진행합니다.

*(※ 로컬 개발이 끝나면 `cd pin-point-infra` 후 `docker-compose down` 명령어로 인프라를 종료할 수 있습니다. 볼륨이 마운트되어 있어 데이터는 날아가지 않습니다.)*

## 📊 인프라 세부 정보 (Infrastructure Details)

로컬 개발 환경과 충돌을 방지하기 위해 기본 포트(3306) 대신 **3307** 포트를 사용합니다. 각 애플리케이션 설정 시 아래 정보를 참고하여 연동해 주세요.

### 1. MySQL (관계형 데이터베이스)

* **Container Name:** `pinpoint-mysql`
* **External Port:** `3307` (Internal: `3306`)
* **Root Password:** `root`
* **Default Database (Schema):** `pin_point_db`
* **Timezone:** `Asia/Seoul`

### 2. Redis (인메모리 캐시 / GeoSpatial)

* **Container Name:** `pinpoint-redis`
* **Port:** `6379`
* **Usage:** 신규 매장 중복 감지, 위치 기반 반경 검색 캐시, 실시간 푸시 알림 데이터 중계

## 🔗 애플리케이션 연동 가이드 (Application Connection)

### ☕ Spring Boot (`application.yml`)

백엔드 프로젝트 개발 시 `src/main/resources/application.yml`에 아래와 같이 데이터 소스를 설정합니다.

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3307/pin_point_db?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: root
    password: root
  data:
    redis:
      host: localhost
      port: 6379

```

### 🐍 Python Crawler (SQLAlchemy / pymysql)

크롤러 프로젝트에서 데이터베이스 및 레디스에 접근할 때 사용할 연결 엔드포인트입니다.

```python
# MySQL Connection URL
DATABASE_URL = "mysql+pymysql://root:root@localhost:3307/pin_point_db"

# Redis Connection
REDIS_HOST = "localhost"
REDIS_PORT = 6379
```