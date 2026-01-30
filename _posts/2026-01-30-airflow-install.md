---
categories:
  - 개발
tags:
  - Data-Engineering
  - Airflow
comments: true
toc: true
title: Airflow 설치하기
Layout: post
---
### Airflow 란?
파이썬을 활용해서 복잡한 데이터 처리 작업을 자동화하는 오픈플랫폼 서비스이다.


### 특징 및 역할
- 워크플로우 오케스트레이션: 여러 작업의 실행 순서와 의존성을 조율할 수 있다.
- DAG(Directed Acyclic Graph): 작업을 순환하지 않는 그래프 형태를 정의하여 데이터가 흐르는 경로를 명확하게 시각화한다.
- Python 기반 코드 작성: 버전 관리, 테스트, 협업에 용이


### 왜 Airflow인가?
- 뛰어난 확장성
- 다양한 외부 서비스 컨트롤 가능
- Airflow에서 제공하는 소스 기반으로 커스터마이징 가능


### Airflow 단점?
- 실시간 워크플로우에는 부적합
- DAG 수가 많아지면 모니터링 어려워짐
- 파이썬이 익숙하지 않으면 다루기 어려움


### 작업 환경
Airflow에 대한 글이기에 앞의 사전 환경 세팅 내용은 아래와 같이 요약한다.
- WSL: 이전에 세팅 해놓은 Ubuntu 환경에서 설치했다.
- UV: 파이썬 + 가상환경 구축에 용이한 환경
- Docker Compose: Ubuntu: Airflow에 쓰이는 Docker 파일들을 WSL에 설치


### Airflow 설치
[Airflow 설치 가이드](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)

1. Docker Compose 다운로드
	```bash
	curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.1.6/docker-compose.yaml'
	```

2. 폴더 사전 세팅
	서비스에 사용되는 폴더 미리 세팅
	```bash
	mkdir -p ./dags ./logs ./plugins ./config
	echo -e "AIRFLOW_UID=$(id -u)" > .env
	```

	문서에는 `AIRFLOW_UID=50000` 나왔으나 실제로는 `AIRFLOW_UID=1000` 이 나왔다.
	이후 config 파일 초기화하는 단계가 문서에 있었으나 선택이라 생략...

3. 데이터베이스 초기화
	초기 실행 시 아래 명령어로 데이터베이스를 초기화한다.
	```bash
	sudo docker compose up airflow-init
	```
	
	초기화 후 `airflow-init-1 exited with code 0` 가 나오면 정상 완료.
	생성된 계정 아이디와 비밀번호는 동일하게 `airflow` 이다.
	(Ubuntu 환경이어서  `sudo`를 앞에 붙였다.)

4. Airflow 띄우기
	```bash
	sudo docker compose up
	```

5. Airflow 페이지 접속
	URL은 `localhost:8080` 에 접속하면 된다.
	![airflow-home-page.png](https://raw.githubusercontent.com/mandel-17/mandel-17.github.io/refs/heads/main/assets/images/airflow-home-page.png)
6. 샘플 DAG 실행
	Airflow에서 제공한 DAG 중 `example_bash_operator`를 실행 해보았다.
	![dag-run-test.png](https://raw.githubusercontent.com/mandel-17/mandel-17.github.io/refs/heads/main/assets/images/dag-run-test.png)
	성공적으로 실행했으며 건너 띈 인스턴스가 2개 있는 것을 확인했다.
	![result-of-dag-run.png](https://raw.githubusercontent.com/mandel-17/mandel-17.github.io/refs/heads/main/assets/images/result-of-dag-run.png)

### 레퍼런스
- [인프런 - Airflow 마스터 클래스](https://www.inflearn.com/course/airflow-%EB%A7%88%EC%8A%A4%ED%84%B0-%ED%81%B4%EB%9E%98%EC%8A%A4/dashboard?cid=331648)
- [Running Airflow in Docker](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)
- [Airflow docker-compose 설치](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)
- [Docker Ubuntu 설치](https://docs.docker.com/engine/install/ubuntu/)