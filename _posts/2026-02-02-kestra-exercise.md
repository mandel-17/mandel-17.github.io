---
Layout: post
title: "[DE zoomcamp] kestra 사용해보기"
toc: true
comments: true
tags:
  - Data-Engineering
  - Airflow
categories:
  - 개발
---
### Kestra 란?
스케쥴링과 이벤트 기반 워크플로우를 쉽게 할 수 있는 오케스트레이션 오픈 폴랫폼이다.


### 특징 및 역할
- 워크플로우 오케스트레이션: 여러 작업의 실행 순서와 의존성을 조율할 수 있다.
- YAML 기반 코드 작성, 노코드 또는 코파일럿과 함께 빌드 할 수 있다.
- 만 개 이상의 플러그인
- 모든 프로그래밍 언어 지원 가능
- 스케쥴 또는 이벤트 기반 트리거

Kestra 관계자가 강의를 해서 그런지 좋은 점만 말한다.

### Kestra 설치
- 설치는 `docker-compose` 파일로 한다.
	- [예제 Docker Compose 파일 바로가기](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/02-workflow-orchestration/docker-compose.yml)

### Kestra 활용하기
실습으로 Kestra를 이용해 데이터 파이프라인을 구축한다.
소스는 뉴욕 택시(`ny_taxi`) 데이터셋을 이용한다.

#### ETL
- **E**xtract: 깃헙에서 데이터를 추출한다.
- **T**ransform: 파이썬으로 데이터를 변환한다.
- **L**oad: Postgres DB에 적재한다.

####  ELT
- **E**xtract: 깃헙에서 데이터를 추출한다.
- **L**oad: 데이터를 데이터레이크인 구글 클라우드 스토리지에 적재한다.
- **T**ransform: 데이터웨어하우스인 빅쿼리에 테이블을 생성하고 데이터 변환을 수행한다.

### GCP 인증 정보 설정 이슈
- GCP 인증 정보 설정하는 데에서 삽질을 했다.
#### 문제
**상황:** 
- kestra에서 GCP Credentials를 어떻게 불러올까?
- 인증을 위해서 제공하는 도커 컴포즈 파일에서 인증 정보를 일반 변수에서 시크릿 변수로 코드 변경 필요했다.

**에러 메시지:**
```
Unterminated string at line 3 column 27 path $.project_id
See https://github.com/google/gson/blob/main/Troubleshooting.md#malformed-json
```

#### 해결 과정
1. kestra Secrets 탭 이동 => 이 기능은 엔터프라이즈 급일 때 사용 가능한 것으로 확인
2. `.env` 파일 활용 => 변수를 불러오지 못함
3. 볼륨 활용 => 파일을 불러오지 못함
4. `.env_encoded`에 인코딩하여 활용 => 위 에러 발생

#### 해결 방법
**해결책:** `.env_encoded`에 인코딩한 값에 따옴표(`''`) 씌움

#### 배운점
- 설명에 나온 것 잘 읽고 하자.
- JSON 형식은 인코딩해도 String 처리가 필요하다.


### 마무리
이번에 Kestra와 다른 오케스트레이션 도구인 Airflow를 비교해보려 했으나 둘 다 처음이다 보니 아직 장단점을 크게 체감하지 못했다. 주로 Airflow가 많이 쓰는 것으로 보여 Airflow 부터 익숙해져 보려고 한다.

### 레퍼런스
[DataTalks Club - Data Engineering Zoomcamp 2주차](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/02-workflow-orchestration)
