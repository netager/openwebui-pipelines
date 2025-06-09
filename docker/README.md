## Dify Docker 배포 Technical Note

본 문서는 Docker Compose를 이용한 Dify 배포를 위한 최신 업데이트 내역, 배포 절차, 마이그레이션 안내 및 환경 변수 설정에 대해 설명합니다.

---

## 1. 업데이트 내역

### 1.1 Certbot 컨테이너  
- `docker-compose.yaml` 파일에 SSL 인증서 관리를 위해 `certbot` 컨테이너가 추가되었습니다.  
- 이 컨테이너는 SSL 인증서를 자동으로 갱신하며, 안전한 HTTPS 연결을 보장합니다.  
> [!info] certbot에 관한 추가 정보는 `docker/certbot/README.md` 참고.

### 1.2 Persistent Environment Variables  
- 환경 변수는 `.env` 파일을 통해 관리됩니다.  
- 이를 통해 배포 간에 구성 값이 지속적으로 유지됩니다.  
> [!info] `.env` 파일은 Docker 및 Docker Compose 환경에서 환경 변수를 중앙 집중식으로 관리하는 파일입니다.[^1]

### 1.3 Unified Vector Database Services  
- 하나의 `docker-compose.yaml` 파일에서 모든 벡터 데이터베이스 서비스를 관리합니다.  
- `.env` 파일 내 `VECTOR_STORE` 환경 변수로 사용하려는 벡터 데이터베이스 (예: `milvus`, `weaviate`, `opensearch`)를 지정할 수 있습니다.

### 1.4 Mandatory .env 파일  
- `docker compose up` 실행 시 반드시 `.env` 파일이 필요합니다.  
- 이 파일에는 배포 구성 및 사용자 지정 설정이 포함됩니다.

### 1.5 Legacy Support  
- 기존 배포 파일은 `docker-legacy` 디렉토리로 이동되었으며, 이후 유지 관리되지 않습니다.

---

## 2. Dify 배포 절차 (`docker-compose.yaml` 이용)

### 2.1 사전 준비 사항
- 시스템에 Docker와 Docker Compose가 설치되어 있어야 합니다.

### 2.2 환경 설정

1. `docker` 디렉토리로 이동합니다.
2. 예제 파일을 복사하여 `.env` 파일을 생성합니다.
   
   ```bash
   cp .env.example .env  # .env 파일 생성
   ```
3. `.env.example` 파일을 참조하여 필요에 따라 `.env` 파일을 수정합니다.

### 2.3 서비스 실행

- `docker` 디렉토리 내에서 다음 명령어를 실행하여 서비스를 시작합니다.
   
   ```bash
   docker compose up
   ```
- 원하는 벡터 데이터베이스를 사용하려면 `.env` 파일 내 `VECTOR_STORE` 변수를 설정합니다.

### 2.4 SSL 인증서 설정

- SSL 인증서를 설정하려면 `docker/certbot/README.md` 파일의 지침을 따르십시오.

---

## 3. 미들웨어 개발 환경 배포 절차

### 3.1 미들웨어 설정

- 미들웨어 서비스(예: 데이터베이스, 캐시 등)를 위해 `docker-compose.middleware.yaml` 파일을 사용합니다.
- `docker` 디렉토리 내에서 `middleware.env` 파일을 생성합니다.
  
   ```bash
   cp middleware.env.example middleware.env  # middleware.env 파일 생성
   ```

### 3.2 미들웨어 서비스 실행

- 다음 명령어를 통해 미들웨어 서비스를 백그라운드 실행합니다.
  
   ```bash
   docker-compose -f docker-compose.middleware.yaml up --env-file middleware.env -d
   ```

---

## 4. 기존 사용자 마이그레이션 안내

### 4.1 변경 사항 숙지
- 새로운 `.env` 구성 및 Docker Compose 설정을 숙지합니다.

### 4.2 사용자 정의 구성 이전
- 기존에 수정한 사항들(`docker-compose.yaml`, `ssrf_proxy/squid.conf`, `nginx/conf.d/default.conf` 등)을 새롭게 생성한 `.env` 파일에 반영합니다.

### 4.3 데이터 마이그레이션
- 데이터베이스, 캐시 등 기존 서비스의 데이터를 백업 및 새로운 구조로 적절히 마이그레이션합니다.

---

## 5. 환경 변수(.env) 개요

### 5.1 주요 모듈 및 사용자 지정

- **Vector Database Services**:  
  - `.env` 파일의 `VECTOR_STORE`에 따라 사용자가 서비스별 엔드포인트, 포트, 인증 정보를 설정할 수 있습니다.

- **Storage Services**:  
  - `STORAGE_TYPE` 변수에 따라 S3, Azure Blob, Google Storage 등의 저장소 설정이 가능합니다.

- **API 및 Web 서비스**:  
  - API 및 웹 프론트엔드 URL 등 서비스 운영에 영향을 주는 설정을 정의합니다.

### 5.2 주요 환경 변수 항목

#### 일반 변수
- `CONSOLE_API_URL`, `SERVICE_API_URL`: API 서비스 URL
- `APP_WEB_URL`: 웹 프론트엔드 URL
- `FILES_URL`: 파일 다운로드 및 미리보기 기본 URL

#### 서버 구성
- `LOG_LEVEL`, `DEBUG`, `FLASK_DEBUG`: 로깅 및 디버깅 설정
- `SECRET_KEY`: 세션 쿠키 및 민감 정보 암호화 키

#### 데이터베이스 구성
- `DB_USERNAME`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`: PostgreSQL 구성

#### Redis 구성
- `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`: Redis 서버 설정

#### Celery 구성
- `CELERY_BROKER_URL`: Celery 메시지 브로커 구성

#### 저장소 구성
- `STORAGE_TYPE`, `S3_BUCKET_NAME`, `AZURE_BLOB_ACCOUNT_NAME`: 파일 저장 옵션 설정

#### 벡터 데이터베이스 구성
- `VECTOR_STORE`: 사용할 벡터 데이터베이스 유형 (예: `weaviate`, `milvus`)
- 각 벡터 스토어에 특화된 설정 (예: `WEAVIATE_ENDPOINT`, `MILVUS_URI`)

#### CORS 구성
- `WEB_API_CORS_ALLOW_ORIGINS`, `CONSOLE_CORS_ALLOW_ORIGINS`: Cross-Origin Resource Sharing 설정

> [!tip] 각 서비스(nginx, redis, db, 벡터 데이터베이스 등)는 `docker-compose.yaml` 파일 내에 참조되는 개별 환경 변수를 가집니다.

---

## 6. 추가 정보

- **Continuous Improvement Phase**:  
  - 커뮤니티의 피드백을 반영하여 배포 프로세스를 지속적으로 개선할 예정입니다.
  
- **Support**:  
  - 자세한 구성 옵션 및 기타 환경 변수 설정은 `.env.example` 파일과 `docker` 디렉토리 내 Docker Compose 구성 파일을 참고하십시오.

---

## 참고문헌

1. [Certbot 공식 문서](https://certbot.eff.org)  
2. [Docker Compose 공식 문서](https://docs.docker.com/compose/)
3. [Dify GitHub Repository](https://github.com/dify-ai/dify)  

[^1]: **`.env` 파일**: Docker 및 Docker Compose 환경에서 컨테이너가 실행 시 접근 가능한 환경 변수들을 중앙집중식으로 관리하는 구성 파일.  
[^2]: **Certbot**: Let’s Encrypt의 SSL 인증서를 자동으로 발급 및 갱신하는 도구.  
[^3]: **Docker Compose**: 여러 컨테이너 애플리케이션을 정의하고 실행하기 위한 도구.  
[^4]: **Middleware**: 애플리케이션의 핵심 기능과 별도로 보조적 기능(예: 데이터베이스, 캐시)을 제공하는 소프트웨어 구성 요소.