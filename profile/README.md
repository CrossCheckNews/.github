# CrossCheckNews — 뉴스 관점 비교 서비스

CrossCheckNews는 동일한 이슈를 서로 다른 국가와 정치적 성향의 언론사가 어떻게 다르게 보도하는지 비교하는 뉴스 관점 비교 서비스입니다.

뉴스 수집, 정규화, 중복 제거, 토픽 클러스터링, 요약 데이터 표시까지 이어지는 파이프라인을 구성하고, 관리자 대시보드에서 파이프라인 실행 상태와 수집 결과를 시각적으로 확인할 수 있도록 구현했습니다.

---

## 프로젝트 구성

| Repository | 설명 |
|---|---|
| `news-bias-api` | Spring Boot 기반 백엔드 API. 뉴스 수집 파이프라인, 토픽/기사 API, 관리자 인증, 대시보드 집계 API 제공 |
| `news-bias-web` | React 기반 프론트엔드. 뉴스 비교 화면, 관리자 대시보드, 파이프라인 실시간 모니터링 UI 제공 |

---

## 핵심 구현 방향

이 프로젝트는 단순 뉴스 목록 서비스가 아니라, 뉴스 데이터를 수집하고 처리하는 과정을 하나의 파이프라인으로 보고 그 실행 상태를 추적할 수 있도록 구성했습니다.

중점적으로 구현한 요소는 다음과 같습니다.

- RSS 기반 뉴스 수집
- URL/헤드라인 정규화
- 중복 기사 제거
- TF-IDF + 코사인 유사도 기반 토픽 클러스터링
- 토픽별 언론사·국가·성향 비교
- 파이프라인 실행 이력 저장
- 관리자 대시보드 지표 시각화
- SSE 기반 실시간 파이프라인 상태 모니터링
- H2 기반 로컬 실행 및 시연용 데이터 자동 적재

---

## 전체 아키텍처

```text
news-bias-web
React + TypeScript + Vite
        │
        │ /api
        ▼
news-bias-api
Spring Boot + JPA + H2
        │
        ├─ RSS Collect
        ├─ Normalize
        ├─ Deduplicate
        ├─ Topic Clustering
        ├─ AI Summary Step
        └─ Pipeline Monitoring
```

프론트엔드는 Vite proxy를 통해 백엔드 API와 연동합니다.

```text

Frontend: http://localhost:5173

Backend:  http://localhost:8080

Proxy:    /api → http://localhost:8080

```
----

## 실행 방법

### 1. Backend 실행

```bash

cd news-bias-api

./gradlew bootRun

```

백엔드는 기본적으로 `http://localhost:8080`에서 실행됩니다.

실행 후 아래 주소에서 API 문서와 H2 콘솔을 확인할 수 있습니다.

| 주소 | 설명 |
|------|------|
| `http://localhost:8080/swagger-ui/index.html` | Swagger UI (전체 API 문서) |
| `http://localhost:8080/h2-console` | H2 콘솔 (JDBC URL: `jdbc:h2:mem:newsdb`) |


관리자 페이지 계정:

```text

ID: admin

PW: admin1234

```

### 2. Frontend 실행

```bash

cd news-bias-web

cp .env.example .env

pnpm install

pnpm run dev

```

브라우저에서 접속합니다.

```text

http://localhost:5173

```

관리자 대시보드:

```text

http://localhost:5173/admin/hello-new-s/dashboard

```

---

## 주요 기능

### 뉴스 비교

- 토픽별 뉴스 비교

- 언론사 국가 및 정치적 성향 표시

- 토픽 상세 화면에서 연결 기사 확인

### 파이프라인 모니터링

- RSS 뉴스 수집 파이프라인 실행

- 파이프라인 실행 이력 저장

- 단계별 실행 상태 추적

- SSE 기반 실시간 파이프라인 상태 모니터링

### 관리자 대시보드

- 수집 기사 수, 토픽 수, 성공/실패 작업 수 요약

- 언론사별 기사 수 차트

- 국가별 토픽 분포 차트

- 파이프라인 결과 상태 차트

- 최근 파이프라인 실행 이력 조회

---

## 기술 스택

### Backend

| 항목 | 내용 |
|------|------|
| Language | Java 21 |
| Framework | Spring Boot |
| ORM | Spring Data JPA |
| Database | H2 |
| RSS Parser | Rome |
| AI | Gemini API 연동 코드 / Ollama fallback 고려 구조 | 
| Auth | JWT |
| Docs | SpringDoc OpenAPI (Swagger UI) |
| Build | Gradle |



### Frontend

| 분류 | 기술 |
|---|---|
| Framework | React, TypeScript |
| Bundler | Vite |
| Routing | React Router |
| Server State | TanStack Query |
| HTTP | Axios |
| Realtime | SSE |
| Chart | Recharts |
| Styling | Tailwind CSS |
| Package Manager | pnpm |

---

## 주요 API

| Method | Path | 설명 |
|---|---|---|
| POST | `/api/v1/auth/login` | 관리자 로그인 |
| POST | `/api/v1/pipeline/collect` | 뉴스 수집 파이프라인 실행 |
| GET | `/api/v1/topics` | 토픽 목록 조회 |
| GET | `/api/v1/topics/{id}` | 토픽 상세 조회 |
| GET | `/api/v1/articles` | 기사 목록 조회 |
| GET | `/api/v1/articles/{id}` | 기사 단건 조회 |
| GET | `/api/v1/publishers` | 언론사 목록 조회 |
| GET | `/api/dashboard/summary` | 대시보드 요약 지표 조회 |
| GET | `/api/dashboard/charts` | 대시보드 차트 데이터 조회 |
| GET | `/api/v1/pipeline/histories` | 파이프라인 실행 이력 조회 |
| GET | `/api/pipeline/stream` | 파이프라인 상태 SSE 스트림 |

---

## 설계 포인트

### 뉴스 도메인과 파이프라인 관측 데이터 분리

뉴스 기사, 토픽, 언론사 데이터와 파이프라인 실행 이력을 별도 테이블로 분리했습니다.

이를 통해 서비스 도메인 데이터와 운영 모니터링 데이터를 독립적으로 관리할 수 있도록 했습니다.

```text

article / topic / publisher

pipeline_run / pipeline_step_history

```

### 중복 제거 기준 다단계화

RSS 피드마다 기사 식별 방식이 다르기 때문에 단일 기준만으로는 중복 제거가 어렵다고 판단했습니다.

```text

Priority 1 — rssGuid + publisherId

Priority 2 — normalizedUrl

Priority 3 — dedupeKey

```

### SSE 기반 실시간 모니터링

파이프라인 상태는 서버에서 클라이언트로 단방향 이벤트를 전달하면 충분한 구조라고 판단했습니다.

따라서 WebSocket 대신 SSE를 사용해 구현 복잡도를 낮추고, HTTP 기반으로 프록시 구성을 단순하게 유지했습니다.

---

## 로컬 실행 설정

본 프로젝트는 과제 리뷰 편의를 위해 H2, 관리자 계정, JWT secret 등 로컬 실행용 기본 설정값을 포함하고 있습니다.

외부 AI API key는 기본 설정에 포함하지 않았으며, 제출용 데모는 저장된 시연용 요약 데이터를 기준으로 확인할 수 있습니다.

운영 환경에서는 API key, JWT secret, DB 접속 정보 등을 환경 변수 또는 별도 secret 관리 방식으로 분리하는 것을 전제로 합니다.

---

## 참고

각 프로젝트의 상세 실행 방법과 구현 설명은 개별 Repository의 README를 참고해주세요.

- `news-bias-api`

- `news-bias-web`

