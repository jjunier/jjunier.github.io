---
title: "[6기] 데브코스 DE WIL 07 | Airflow 기반 데이터 파이프라인"
tags:
    - DevCourse
    - Data Engineering
    - Airflow
date: "2025-05-23"
thumbnail: "/assets/img/thumbnail/devcourse.png"
bookmark: true
---

## 이번 주 학습 목표 
--- 
- Airflow의 DAG, Task, Operator 구조와 실행 흐름(start_date, execution_date, catchup)을 정확히 이해한다.
- ETL과 ELT, Full Refresh와 Incremental Update의 차이를 구분하고 운영 관점에서의 장단점을 설명할 수 있다.
- 트랜잭션, 멱등성, Backfill을 고려하여 안정적인 데이터 파이프라인을 설계할 수 있다.

## 데이터 파이프라인 이해하기
---
**데이터 파이프라인(Data Pipeline)**이란 간단하게 말하자면, **데이터 소스(Data Source)**로부터 **목적지(Destination)**로 이동시키는 일련의 작업을 일컫는 말이며, 이 작업은 대부분 `코드(Python, Scala)`나 `SQL`을 통해 구현되며, 대규모 데이터의 경우 `Spark`와 같은 분산 처리 엔진이 활용되기도 한다.

**데이터의 소스**는 매우 다양하며, 대표적인 데이터 소스로는 클릭 스트림 데이터, 광고 성과 데이터, 트랜잭션 로그, 센서 데이터, 메타 데이터 등 비즈니스 활동 전반에서 생서오디는 모든 데이터가 출발점이 될 수 있다.

**목적지** 또한 다양하지만, 대부분의 경우엔 `데이터 웨어하우스(Data Warehouse)`가 그 중심이 된다. 그 외에도 NoSQL 스토리지, S3와 같은 오브젝트 스토리지 혹은 프로덕션 데이터베이스가 목적지가 되기도 한다.

결국엔 데이터 파이프라인은 **"데이터를 어디서 가져와서, 어떻게 가공하며, 어디에 쓸 것인가"를 정의하는 구조**로 볼 수 있다.

### ETL이란?
---
ETL은 **Extract, Transform, Load**의 약자로, 데이터 엔지니어링에서 가장 전통적인 개념이다. 먼저 데이터를 `추출(Extract)`하고, 외부 환경에서 필요한 형태로 `변환(Transform)`한 뒤, 최종 목적지에 `적재(Load)`한다.

ETL 방식에서 변환 작업은 데이터 웨어하우스 외부에서 이루어지는데, 즉 데이터를 가공하는 로직이 별도의 처리 레이어에 존재하며, 완성된 결과만을 데이터 웨어하우스에 적재한다.

가장 주로 사용되는 `Airflow`에서는 이러한 ETL의 각 단계를 개별 **Task로 정의**하고 이를 DAG 형태로 연결하여 실행 순서를 제어한다. 관련 예시로 **“API로부터 데이터 추출 → Spark로 변환 → 데이터 웨어하우스 적재”**라는 일련의 흐름을 하나의 DAG로 표현할 수 있다.


### ELT란?
---
최근에는 ELT 방식이 널리 사용되고 있다. ELT는 Extract, Load, Transform의 순서를 따르며, **데이터를 원형 그대로 웨어하우스에 적재**한 뒤, **웨어하우스 내부에서 SQL을 활용해 변환 작업을 수행**하는 방식이다.

해당 방식의 핵심은 **“변환을 웨어하우스 내부에서 수행한다”**는 점이다. 클라우드 데이터 웨어하우스의 성능이 크게 향상되면서, 굳이 외부에서 복잡하게 가공하지 않고 내부 연산 능력을 활용하는 것이 더 효율적인 경우가 많아졌기 때문이다.

ELT는 **데이터 분석가**가 주도하는 경우가 많으며, 원시 데이터를 기반으로 **요약 테이블이나 리포트용 테이블을 생성**하고, 이를 **BI 도구에서 활용**하는 데에 사용된다. 이러한 영역을 전문적으로 다루는 대표적인 도구로 `dbt`가 있으며, dbt는 S**QL을 기반으로 데이터 모델을 정의하고 관리**하는 도구이다.

ETL이 데이터 엔지니어 중심의 **외부 가공 모델**이라면, ELT는 데이터 분석가 중심의 **내부 가공 모델**이라고 이해할 수 있다.

### Data Lake와 Data Warehouse
---

![Data Lake & Data Warehouse](/assets/img/contents/datalake_datawarehouse.png "Data Lake & Data Warehouse")

데이터 아키텍처를 논할 때 빠지지 않는 개념이 `Data Lake`와 `Data Warehouse`이다. 두 개념은 목적과 활용 방식에서 뚜렷한 차이를 가진다.

**Data Lake**
Data Lake는 **구조화 데이터와 비구조화 데이터를 모두 저장할 수 있는 대규모 스토리지**로, 로그 파일, 이미지, JSON, 스트림 데이터 등 **원본 형태 그대로의 데이터를 보존**하는 것이 특징이다.

**“모든 데이터를 일단 저장해두는 공간”**이라고 정의 가능하며, 필요에 따라 정제하여 데이터 웨어하우스로 이동시키거나, 레이크 위에서 직접적으로 분석 작업을 수행하기도 한다.

**Data Warehouse**
Data Warehouse는 **정제되고 구조화된 데이터를 저장하는 공간**으로, 보존 정책이 존재하며, 분석과 리포팅에 최적화되어 있다. 스타 스키마나 Snowflake 스키마와 같은 모델링 기법이 적용되며, 성능과 일관성이 중요한 요소다.

### 데이터 파이프라인의 세 가지 유형
---
1.Raw ETL Job
**외부 API나 내부 데이터 소스로부터 데이터를 수집하고, 필요한 포맷으로 변환한 뒤 데이터 웨어하우스에 적재하는 작업**으로, 데이터의 규모가 커질수록 Spark와 같은 분산 처리 기술이 필요해진다. 

이 유형은 전통적인 **데이터 엔지니어의 역할**에 해당한다.

2.Summary & Report Jobs
**이미 웨어하우스나 레이크에 존재하는 데이터를 읽어 다시 웨어하우스에 요약 테이블 형태로 저장하는 작업**으로, 일별 매출 요약 테이블, 사용자 리텐션 테이블, A/B 테스트 결과 테이블 등이 이에 해당한다. 

데이터 엔지니어의 관점에서는 분석가가 이런 작업을 효율적으로 수행할 수 있는 환경을 제공하는 것이 중요하다. 이 지점에서 dbt와 같은 도구가 중요한 역할을 한다.

3.Production Data Jobs
**웨어하우스에 저장된 데이터를 다시 외부 스토리지나 프로덕션 환경으로 내보내는 작업**으로, 성능상의 이유로 요약 정보를 Redis에 적재하거나, 머신러닝 모델에서 사용할 피처를 미리 계산해 NoSQL 스토리지에 저장하는 경우가 이에 해당한다. 

Cassandra, HBase, DynamoDB와 같은 NoSQL, MySQL과 같은 OLTP 데이터베이스, Redis나 Memcache, ElasticSearch 등이 주요 타겟이 된다.

## 데이터 파이프라인 설계 시 고려사항
---
데이터 파이프라인을 처음 만들 때 흔히 이런 기대를 한다.

>> **“내가 만든 파이프라인은 문제없이 동작할 것이다.”
  “운영과 관리는 크게 어렵지 않을 것이다.”**

하지만 현실은 다르다. 파이프라인은 다양한 이유로 실패한다.

단순한 버그부터, **데이터 소스의 장애, API 포맷 변경, 예상하지 못한 스키마 변경**까지 원인은 다양하다. 
또한 **여러 파이프라인 간 의존성**을 충분히 이해하지 못한 상태에서 설계하면, **한 소스의 실패가 연쇄적인 장애**로 이어질 수 있다.

파이프라인 수가 늘어날수록 유지보수 비용은 기하급수적으로 증가하는데, 마케팅 채널 데이터가 업데이트되지 않으면, 이를 참조하는 모든 리포트 테이블이 함께 멈출 수 있다. 

또한, 관리해야 할 테이블 수가 늘어나면서 source of truth 혼란, 검색 비용 증가 등의 문제도 발생한다.

### Best Practice 1: Full Refresh vs Incremental
---
가능하다면 데이터가 크지 않을 경우 Full Refresh 전략이 가장 단순하고 안전하다. 매 실행 시 전체 데이터를 다시 만들어 정합성을 보장하는 방식이다. Incremental 업데이트가 필요한 경우에는 데이터 소스가 몇 가지 조건을 만족해야 한다:

- 프로덕션 DB 테이블이라면 최소한 modified, deleted 필드가 존재해야 한다.
- API 기반 소스라면 특정 시점 이후 생성되거나 수정된 레코드를 조회할 수 있어야 한다.

증분 기준이 불명확하면 데이터 누락이나 중복이 발생하기 쉽다.

### Best Practice 2: 멱등성(Idempotency)
---
멱등성은 데이터 파이프라인 설계에서 가장 중요한 개념 중 하나다.

**같은 입력 데이터를 가지고 여러 번 실행해도 결과 테이블이 달라지지 않아야 한다.** 중복 레코드가 생기거나, 실행 횟수에 따라 결과가 변하면 안 된다.

이를 위해 중요한 처리 구간은 **하나의 atomic action으로 실행**되어야 한다. SQL 기반 처리라면 **transaction을 활용해 원자성을 보장**하는 것이 기본이다.

### Best Practice 3: 재실행과 Backfill
---
실패는 반드시 발생한다. 중요한 것은 **쉽게 재실행할 수 있어야 한다는 점**이다.

또한 **과거 데이터를 다시 채워야 하는 상황(Backfill)**도 자주 발생한다. 이때 **날짜 단위로 재처리**가 가능하도록 설계되어 있어야 한다.

Airflow는 스케줄 기반 실행과 과거 실행(backfill)에 강점을 가진다. 하지만 설계가 이를 고려하지 않았다면 도구의 장점도 활용하기 어렵다.

### Best Practice 4: 명확한 입력/출력과 문서화
---
모든 데이터 파이프라인은 다음을 명확히 해야 한다:

- **입력 데이터는 무엇인가**
- **출력 테이블은 무엇인가**
- **비즈니스 오너는 누구인가**

“누가 이 데이터를 요청했는가”를 기록으로 남겨야 한다. 이는 이후 데이터 카탈로그에 반영되어 데이터 디스커버리와 데이터 리니지 관리에 활용될 수 있다.

데이터 리니지를 이해하지 못하면, 테이블 하나를 수정하는 순간 예상치 못한 장애를 유발할 수 있다.

### Best Practice 5: 불필요한 데이터 정리
---
사용하지 않는 테이블과 파이프라인은 적극적으로 제거해야 한다:

- **Unused 테이블 삭제**
- **더 이상 쓰이지 않는 DAG 제거**
- **오래된 데이터는 Data Lake나 저비용 스토리지로 이동**

데이터 웨어하우스에는 반드시 필요한 데이터만 남기는 것이 운영 비용과 복잡도를 줄이는 방법이다.

### Best Practice 6: 사고 리포트(Post-mortem)
---
데이터 사고가 발생할 때마다 Post-mortem을 작성하는 것이 좋다. 목적은 비난이 아니라 재발 방지다:

- **Root cause 분석**
- **재발 방지를 위한 액션 아이템 정의**
- **기술 부채 수준 점검**

Post-mortem은 조직의 데이터 성숙도를 보여주는 지표이기도 하다.

### Best Practice 7: 입력과 출력 검증
---
중요 파이프라인은 최소한의 검증 로직을 포함해야 한다:

- **입력 레코드 수와 출력 레코드 수 비교**
- **Primary Key가 존재한다면 uniqueness 체크**
- **중복 레코드 여부 확인**
- N**ull 비율 급증 여부 점검**

간단한 체크만으로도 많은 사고를 예방할 수 있다.

## Airflow 소개
---

![Airflow](/assets/img/contents/airflow.png "Airflow")

`Apache Airflow` 는 파이썬으로 작성된 **데이터 파이프라인(ETL) 오케스트레이션 프레임워크**이다. 

Airbnb에서 시작되어 현재는 아파치 오픈소스 프로젝트로 발전했으며, 데이터 파이프라인 관리 도구 중 가장 널리 사용되는 표준 중 하나로 자리 잡았다.

Airflow의 핵심 목적은 **데이터 파이프라인을 코드로 정의하고, 스케줄링하고, 모니터링하는 것**이다. **정해진 시간에 ETL을 실행**하거나, **하나의 작업이 끝난 뒤 다음 작업을 실행하도록 의존성을 설정**할 수 있다. 또한 웹 UI를 통해 실행 상태를 시각적으로 확인할 수 있다.

Airflow에서는 데이터 파이프라인을 `DAG(Directed Acyclic Graph)` 라고 부른다. **하나의 DAG는 여러 개의 Task로 구성되며, Task 간 의존성을 정의**할 수 있다.

### Airflow 주요 기능
---
Airflow는 데이터 파이프라인을 쉽게 만들 수 있도록 다양한 모듈을 제공한다. **여러 데이터 소스와 데이터 웨어하우스를 연결할 수 있는 Operator와 Hook**이 기본적으로 제공되며, 확장도 용이하다.

또한 운영 측면에서 중요한 기능들을 포함한다. 대표적으로 **Backfill** 기능이 있다. **특정 기간의 과거 데이터를 다시 실행해야 할 때, 날짜 단위로 손쉽게 재처리**할 수 있다. 이는 배치 기반 데이터 환경에서 매우 중요한 기능이다.

### Airflow 구성 요소
---
Airflow는 다음과 같이 다섯 가지 컴포넌트로 구성된다:

1. `Web Server`: Flask 기반으로 구현되어 있으며, DAG 상태와 실행 결과를 시각화한다.
2. `Scheduler`: DAG를 읽고 실행 시점을 판단해 작업을 배정한다.
3. `Worker`: 실제 Task를 실행한다.
4.` Metadata Database`: DAG 실행 이력과 상태 정보를 저장한다. (기본값: SQLite, 프로덕션 추천: MySQL, Postgres)
5. `Queue`: 다중 서버 환경에서 Worker들에게 작업을 분배할 때 사용된다.

**단일 서버 구조**

![Single Server Structure](/assets/img/contents/single_server.png "Single Server Structure")

단일 서버 구성에서는 **Scheduler, Worker, Web Server가 하나의 서버에서 동작**한다. 소규모 환경이나 개발 환경에 적합한 구조다.

**다중 서버 구조**

![Multiple Server Structure](/assets/img/contents/multiple_server.png "Multiple Server Structure")

트래픽과 DAG 수가 증가하면 **스케일링이 필요**하다. 방법은 두 가지로 정의할 수 있다:

- `스케일 업`: 더 좋은 사양의 서버 사용
- `스케일 아웃`: Worker 서버를 추가

다중 서버 환경에서는 **Queue가 추가**되고, **Executor 종류에 따라 작업 분배 방식**이 달라진다.

### Executor 종류
--- 
Airflow는 다양한 Executor를 지원한다.

![Executor](/assets/img/contents/executor.png "Executor")

- Sequential Executor
- Local Executor
- Celery Executor
- Kubernetes Executor
- CeleryKubernetes Executor
- Dask Executor

**Executor는 Task를 어떤 방식으로 실행할지를 결정하는 핵심 컴포넌트**이다. 단일 머신에서 실행할지, 분산 큐 기반으로 실행할지, Kubernetes 위에서 실행할지에 따라 선택이 달라진다.

### Airflow 개발의 장단점
---
`장점`
- 데이터 파이프라인을 코드로 세밀하게 제어 가능함
- 다양한 데이터 소스와 웨어하우스를 지원함
- Backfill이 용이함

`단점`
- 러닝 커브가 존재함
- 개발 환경 구성이 쉽지 않음
- 직접 운영 시 인프라 관리 부담이 커, 클라우드용 버전 사용이 선호됨

## DAG란 무엇인가
---
**Apache Airflow**에서 데이터 파이프라인은 `DAG(Directed Acyclic Graph)` 라는 개념으로 표현된다. DAG는 말 그대로 **방향성이 있고(Directed), 순환이 없는(Acyclic) 그래프(Graph)**를 의미한다.

**Airflow에서 하나의 ETL은 곧 하나의 DAG이며, DAG는 하나 이상의 Task로 구성**된다. 예를 들어 가장 단순한 형태라면 Extract → Transform → Load의 세 단계로 구성될 수 있다.

### Task와 Operator
---
Airflow에서 `Task`는 **Operator를 통해 생성**된다. 여기서 `Operator`는 **“어떤 작업을 수행할 것인가”를 정의하는 실행 단위**이며, Airflow는 다양한 기본 Operator를 제공한다. 

예를 들면 다음과 같다.

- **Postgres 쿼리 실행**
- **Redshift 적재**
- **S3 읽기/쓰기**
- **Hive 쿼리**
- **Spark Job 실행**
- **Shell Script 실행**

상황에 맞는 Operator를 선택해 사용하거나, 필요하다면 직접 커스텀 Operator를 개발할 수도 있다. 결국 DAG는 **“Operator로 만들어진 Task들의 집합”**이라고 말할 수 있다.

### DAG 정의 시 필요한 기본 정보
---
모든 DAG에는 공통적으로 필요한 기본 설정이 있는데, 다음과 같은 예시로 정의할 수 있다:

```python
default_args = {
    'owner': 'jun',                                         # DAG의 소유자
    'start_date': datetime(2020, 8, 7, hour=0, minute=0),   # 실행 범위(시작일)
    'end_date': datetime(2020, 8, 31, hour=23, minute=0),   # 실행 범위(종료일)
    'email': ['yyt1186@gmail.com'],
    'retries': 1,                                           # 실패 시 재시도 횟수
    'retry_delay': timedelta(minutes=3),                    # 재시도 간격
}
```

DAG의 객체는 다음과 같은 방식으로 생성할 수 있다:

```python
from airflow import DAG

test_dag = DAG(
    "dag_v1",          # DAG 이름
    schedule="0 9 * * *",
    tags=['test'],
    default_args=default_args
)
```

여기서 중요한 것은 schedule으로, Airflow는 크론탭(Crontab) 문법을 따른다.

- "0 * * * *" → 매 정각마다 실행
- "0 12 * * *" → 매일 12:00에 실행

즉, DAG는 단순한 코드 묶음이 아니라 **“언제 실행될 것인가”까지 정의된 스케줄 단위**이다.

### Operator 생성 예시 1
---

```python
from airflow.operators.bash_operator import BashOperator

t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=test_dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=test_dag)

t3 = BashOperator(
    task_id='ls',
    bash_command='ls /tmp',
    dag=test_dag)
```

Task 간의 의존성은 다음과 같이 정의할 수 있다.

```python
t1 >> t2
t1 >> t3
t1 >> [t2, t3]

t2.set_upstream(t1)
t3.set_upstream(t1)
```

`>>` 연산자는 **“앞의 Task가 끝난 후 뒤의 Task가 실행된다”**는 의미이다.

### Operator 생성 예시 2: 시작과 종료 노드 추가
---

```python
from airflow.operators.bash_operator import BashOperator
from airflow.operators.dummy_operator import DummyOperator

start = DummyOperator(dag=dag, task_id="start")
t1 = BashOperator(
    task_id='t1',
    bash_command='ls /tmp/downloaded',
    retries=3,
    dag=dag)

t2 = BashOperator(
    task_id='t2',
    bash_command='ls /tmp/downloaded',
    dag=dag)

end = DummyOperator(dag=dag, task_id='end')
```

의존성은 다음과 같이 정의할 수 있다.

```python
start >> t1 >> end
start >> t2 >> end
start >> [t1, t2] >> end
```

## 트랜잭션이란 무엇인가
---
데이터 파이프라인이나 데이터베이스 작업을 하다 보면, **“중간에 실패하면 안 되는 작업”**을 반드시 만나게 된다.
예를 들어, 은행 이체 과정을 생각해보자.

1. 내 계좌에서 인출
2. 다른 사람 계좌로 송금

만약 인출은 성공했지만 송금 단계에서 오류가 발생한다면 어떻게 될까?
내 돈은 빠져나갔지만 상대방은 받지 못하는 불완전한 상태가 된다.

이처럼 여러 단계가 **하나의 논리적 작업**으로 묶여 있고, 중간에 실패하면 전체를 되돌려야 하는 경우에 필요한 개념이 바로 **트랜잭션(Transaction)** 이다.

### 트랜잭션의 개념
---
트랜잭션은 여러 개의 SQL을 **하나의 원자적(Atomic) 작업처럼 처리하는 방법**이다. 일반적인 형태는 다음과 같다.

```sql
BEGIN;

SQL 1;
SQL 2;
SQL 3;

COMMIT;
```

- `BEGIN` : 트랜잭션 시작
- `COMMIT` : 모든 작업이 성공했을 때 최종 반영
- `ROLLBACK` : 중간에 하나라도 실패하면 이전 상태로 복구

트랜잭션 내부에서 실행된 SQL의 결과는 **임시 상태**로 존재한다.
COMMIT이 되기 전까지는 다른 세션에서 보이지 않는다.

중간에 문제가 발생하면 다음과 같이 처리한다.

```sql
BEGIN;

SQL 1;
SQL 2;

ROLLBACK;
```
ROLLBACK이 실행되면 BEGIN 이전 상태로 돌아간다.

### 트랜잭션의 동작 방식
---
트랜잭션 안에 포함된 SQL은 모두 성공해야만 최종 상태로 확정된다.

- 모두 성공 → `COMMIT`
- 하나라도 실패 → `ROLLBACK`

이 특성 덕분에 데이터 정합성을 유지할 수 있다.

다만, 트랜잭션에 포함되는 SQL은 최소화하는 것이 좋다.
트랜잭션이 길어질수록 락(lock) 유지 시간이 길어지고, 성능과 동시성에 영향을 줄 수 있기 때문이다.

### Autocommit과 트랜잭션
---
트랜잭션에는 두 가지 동작 방식이 있다.

1. autocommit=True

- 각 SQL 실행 시마다 자동으로 COMMIT
- 모든 변경이 즉시 물리 테이블에 반영됨

이 상태에서 트랜잭션으로 묶고 싶다면 명시적으로 BEGIN과 COMMIT을 사용해야 한다.

```sql
BEGIN;
...
COMMIT;
```

또는 실패 시 ROLLBACK을 호출한다.

2. autocommit=False

- 기본적으로 모든 SQL이 자동 커밋되지 않음
- 명시적으로 commit/rollback을 호출해야 반영됨

Python에서는 보통 다음과 같이 처리한다.

```sql
try:
    cur.execute(sql1)
    cur.execute(sql2)
    conn.commit()
except Exception:
    conn.rollback()
    raise
```

성공하면 commit(), 실패하면 rollback()을 실행한다.

### try/except 사용 시 주의할 점
---
다음과 같은 코드가 있다고 가정하자.

```sql
try:
    cur.execute(create_sql)
    cur.execute("COMMIT;")
except Exception as e:
    cur.execute("ROLLBACK;")
```

이 경우 예외가 발생해도 에러가 외부로 전달되지 않을 수 있다.
ETL 관점에서는 에러가 “숨겨지는 것”이 가장 위험하다.

따라서 반드시 raise를 사용해 원래 예외를 다시 던져주는 것이 좋다.

```sql
try:
    cur.execute(create_sql)
    cur.execute("COMMIT;")
except Exception as e:
    cur.execute("ROLLBACK;")
    raise
```
이렇게 해야:

- 트랜잭션은 롤백되고
- 에러는 상위 레벨(Airflow 등)로 전달되며
- DAG 실행이 실패로 기록된다

데이터 파이프라인에서는 **조용히 실패하는 것보다 명확히 실패하는 것이 훨씬 안전하다.**

## Backfill과 Airflow
---
Incremental Update 기반의 데이터 파이프라인은 효율적이지만, 운영 난이도가 높다. 
특히 **실패한 날짜를 어떻게 재실행할 것인가**는 데이터 엔지니어의 삶의 질에 직접적인 영향을 준다.

### Full Refresh vs Incremental
---
가능하다면 Full Refresh가 가장 단순하다.

- 문제가 생기면 전체를 다시 실행하면 된다.
- 정합성 측면에서도 안전하다.

반면 Incremental Update는:

- 효율성은 좋지만
- 실수로 특정 날짜 데이터가 빠질 수 있고
- 과거 데이터를 다시 채우려면 별도 작업이 필요하다.

즉, Incremental 방식에서는 **재실행(Backfill)이 얼마나 쉬운 구조인가**가 핵심이다.

### 잘못 설계된 Daily ETL의 예
---
보통 다음과 같이 구현하는 경우가 많다.

```python
from datetime import datetime, timedelta

y = datetime.now() - timedelta(1)
yesterday = datetime.strftime(y, '%Y-%m-%d')

sql = f"SELECT * FROM table WHERE DATE(ts) = '{yesterday}'"
```

문제는, 1년 치 데이터를 다시 채워야 한다면?

- 코드를 수정해야 한다.
- 날짜를 하드코딩한다.
- 실수하기 쉽다.
- 운영 중 코드 변경은 위험하다.

이 방식은 Backfill에 매우 취약하다.

### Airflow의 접근 방식
---
Apache Airflow 는 이 문제를 시스템적으로 해결한다. 핵심 개념은 execution_date다.

- 모든 DAG 실행에는 execution_date가 존재한다.
- 이 값은 “읽어와야 하는 데이터의 날짜”다.
- 실행 결과는 Metadata DB에 기록된다.

즉, 데이터 엔지니어는 날짜를 계산하지 않는다. Airflow가 지정해준 execution_date를 그대로 사용하면 된다.

```python
sql = f"""
SELECT *
FROM table
WHERE DATE(ts) = '{{{{ ds }}}}'
"""
```

이렇게 작성하면, Backfill이 자동으로 쉬워진다.

### Daily Incremental Update의 시간 개념

예를 들어: 

- 2020-11-07 데이터부터 매일 하루치씩 읽는다고 가정
- 이 ETL은 언제 처음 실행되어야 할까?

정답은 2020-11-08이다.

하지만 읽어야 할 데이터는? → 2020-11-07

여기서 중요한 점:

- start_date는 DAG가 시작되는 날짜가 아니라 **처음 읽어야 할 데이터의 날짜**
- execution_date는 읽어야 할 데이터의 날짜

| 구분            | 의미              |
| -------------- | ----------------- |
| start_date     | 처음 읽을 데이터의 날짜 |
| execution_date | 해당 실행이 처리할 데이터 날짜 |

### catchup의 위험성: 만불짜리 쿼리
---
잘못 설정된 start_date + catchup=True + 대용량 쿼리 조합은 매우 위험하다.

예:

- start_date = 2020-08-06
- 오늘 날짜 = 2020-08-14
- catchup = True (기본값)

이 경우 DAG를 Enable하는 순간:

- 8번 실행된다.
- 만약 BigQuery/Snowflake에서 2천불짜리 쿼리라면?
- 8번 실행 → 1만6천불

Redshift처럼 월 정액 구조는 괜찮지만, 쿼리 단위 과금 시스템에서는 치명적이다.

### start_date & execution_date 이해 문제
---
조건:

- start_date = 2020-08-10 02:00:00
- daily job
- catchup = True
- 현재 시간 = 2020-08-13 20:00:00
- 지금 처음 활성화

이 경우 실행 횟수는?

- 2020-08-10 02:00:00
- 2020-08-11 02:00:00
- 2020-08-12 02:00:00
- 2020-08-13 02:00:00

총 4번 실행되며, 이것이 catchup의 기본 동작이다.

### Backfill 관련 핵심 파라미터
---

| 변수             | 설명  |
| -------------- | ------------------------------------------------- |
| start_date     | DAG가 처음 읽어야 할 데이터의 날짜 |
| execution_date | 해당 실행이 처리할 데이터의 날짜 |
| catchup        | start_date 이후 실행되지 않은 구간을 자동으로 따라잡을지 여부 (기본 True) |
| end_date       | 특정 기간까지만 실행하고 싶을 때 사용 |

수동 Backfill 명령으로 특정 날짜 범위를 재실행할 수 있다.

```bash
airflow dags backfill -s 2023-01-01 -e 2023-01-31 dag_id
```