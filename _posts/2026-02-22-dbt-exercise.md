---
Layout: post
title: "[DE zoomcamp] dbt 사용해보기"
toc: true
comments: true
tags:
  - Data-Engineering
  - BigQuery
categories:
  - 개발
---

뉴욕 택시 데이터를 BigQuery나 DuckDB에 적재하고 dbt로 정제하는 과정을 배웠다.

### 1. dbt란?

**데이터 변환(Transformation) 워크플로우 도구**입니다.

데이터 웨어하우스 위에 올라가서, 원시 데이터를 분석/BI/ML에 쓸 수 있는 깔끔한 데이터로 변환합니다.

#### 1.1. 핵심 개념

- SQL `SELECT`만 작성하면 dbt가 `CREATE TABLE` 등 나머지를 알아서 처리
- 의존성 순서대로 자동 실행, 결과를 테이블/뷰로 저장

#### 1.2. 해결하는 문제

기존 SQL 변환 작업에 **소프트웨어 엔지니어링 관행**을 도입:
- **버전 관리** — git으로 SQL 코드 관리
- **모듈화** — 복잡한 쿼리를 재사용 가능한 조각으로 분리
- **테스트** — 자동화된 데이터 품질 검증
- **문서화** — 코드에서 자동 생성
- **환경 분리** — dev/prod 독립적으로 운영

#### 1.3. dbt Core vs dbt Cloud

|         | dbt Core          | dbt Cloud           |
| ------- | ----------------- | ------------------- |
| 형태      | 오픈소스, 로컬 설치       | SaaS (유료/무료 플랜)     |
| 오케스트레이션 | 직접 구성 (Airflow 등) | 내장 스케줄러             |
| 문서 호스팅  | 직접 구성             | 자동 제공               |
| 이 과정    | Option B (DuckDB) | Option A (BigQuery) |

### 2. 데이터 구조
데이터 구조는 3가지 단계를 걸쳐서 이루어 진다. 

##### staging
- 소스 정의: 로우 데이터에 대한 정의
- 스테이징 모델: 최소한의 정제로 데이터를 각각 테이블에 1 대 1 복사하기

##### intermeidate
- 복잡한 조인 수행
- 더러운 데이터 클린징 또는 정규화

##### marts
- 최종적으로 사용할 수 있는 데이터 존재
- 대쉬보드를 통해 유저한테 제공할 데이터


### 3. dbt 주요 커맨드
#### 3.1. 초기 설정 명령어

|명령어|설명|
|---|---|
|`dbt init`|프로젝트 디렉토리 구조 생성 (최초 1회)|
|`dbt debug`|DB 연결 및 `profiles.yml` 유효성 검사|
|`dbt deps`|`packages.yml`의 패키지 설치|
|`dbt clean`|`target/`, `dbt_packages/` 등 빌드 산출물 삭제|

#### 3.2. 기능별 명령어

| 명령어                       | 설명                           |
| ------------------------- | ---------------------------- |
| `dbt seed`                | `seeds/` 폴더의 CSV를 DB에 로드     |
| `dbt snapshot`            | SCD Type 2 방식으로 데이터 변경 이력 추적 |
| `dbt source freshness`    | 소스 데이터 최신 여부 확인              |
| `dbt docs generate/serve` | 문서 생성 및 로컬 웹서버로 확인           |

#### 3.3. 핵심 명령어

| 명령어           | 설명                                                       |
| ------------- | -------------------------------------------------------- |
| `dbt compile` | Jinja/ref 해석 후 순수 SQL만 `target/compiled/`에 출력 (DB 접근 없음) |
| `dbt run`     | 모든 모델을 의존성 순서대로 빌드                                       |
| `dbt test`    | 데이터 품질 테스트 실행                                            |
| `dbt build` ⭐ | run + test + seed + snapshot 통합 실행 (실패 시 하위 모델 자동 스킵)    |
| `dbt retry`   | 이전 실패 지점부터 재실행 (`run_results.json` 활용)                   |

#### 3.4. 주요 플래그

| 플래그                        | 설명                          |
| -------------------------- | --------------------------- |
| `--full-refresh`           | incremental 모델을 전체 재빌드      |
| `--fail-fast`              | 경고도 에러로 처리, 즉시 중단           |
| `--target prod`            | 실행 환경 지정 (기본값: dev)         |
| `--select stg_green`       | 특정 모델만 선택                   |
| `--select +model`          | 해당 모델 + 모든 상위 의존성 포함        |
| `--select model+`          | 해당 모델 + 모든 하위 의존성 포함        |
| `--select state:modified+` | 변경된 모델 + 하위만 실행 (CI/CD에 유용) |

### 삽질 포인트
#### 프로파일 오류
- 제공된 `.dbt` 의 `profiles.yml` 에서 `dev`로 설정 되어 있어서 에러가 발생했다. `target`을 `prod`로 수정했다.
	```
	taxi_rides_ny:
	  target: dev -> prod로 수정!
	  outputs:
	...
	```


### 결론 + 다음 스텝
dbt를 통해 데이터를 조작하여 원하는 뷰를 만들 수 있다는 것을 배웠다.
다음에 데이터를 다루는 종합 플랫폼인 bruin에 대해 정리해보겠다.