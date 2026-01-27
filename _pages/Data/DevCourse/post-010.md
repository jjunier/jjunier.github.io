---
title: "[6기] 데브코스 DE WIL 10 | DBT, 데이터 디스커버리"
tags:
    - DevCourse
    - Data Engineering
    - DBT
date: "2025-06-04"
thumbnail: "/assets/img/thumbnail/devcourse.png"
bookmark: true
---

## 이번 주 학습 목표 
---
- dbt의 핵심 개념과 구성 요소(models, materialization, tests, snapshots, docs)를 이해하고, ELT 기반 데이터 파이프라인을 구조적으로 설계할 수 있다.
- Fact·Dimension 모델링, 증분 로드, 변경 이력 관리(SCD Type 2), 데이터 품질 테스트를 통해 신뢰성 있는 분석용 데이터셋을 구축할 수 있다.
- dbt 문서화와 데이터 카탈로그 개념을 활용해 데이터 리니지, 메타데이터, 거버넌스를 체계적으로 관리하는 방법을 이해한다.

## ELT의 미래는?
---
ETL을 하는 이유는 결국 ELT를 하기 위함이며 이때 데이터 품질 검증이 중요해진다.

**데이터 품질의 중요성 증대**
- 입출력 체크
- 더 다양한 품질 검사 
- 리니지 체크
- 데이터 히스토리 파악

따라서 데이터 품질을 유지하는 것은, 비용/노력 감소와 생산성 증대의 지름길이다. 이러한 문제를 해결하기 위해서 나온 tool이 `DBT(Data Build Tool)`이다.

## Database Normalization
---
`데이터 정규화(Data Normalization)`을 하는 이유는 데이터베이스를 좀 더 조직적이고 일관된 방법으로 디자인함으로써 유지보수를 더 쉽게 하기 위해서 진행한다. 이는 데이터베이스의 정합성을 쉽게 유지하고, 레코드들을 수정/적재/삭제를 용이하게 하는 것을 의미한다.

**Normalization에 사용되는 개념**
- Primary Key (기본키)
- Composite Key (복합키)
- Foreign Key (외래키)


### 제 1 정규형(1NF; First Normal Form)
---
제 1 정규형은 한 셀에 하나의 값만 있어야 하는 `원자성(Atomicity)`을 만족하는 정규형이다. Primary Key가 있어야 하며, 중복된 키나 레코드들이 없어야 한다. 결국, 해당 정규형의 목표는 **중복을 제거하고 원자성을 만족**하는 것이다.

**1NF 테이블 예제 - Employee**

|EMPLOYEE_ID|NAME|JOB_CODE|JOB|STATE_CODE|HOME_STATE|
|---|---|---|---|---|---|
|E001|Alice|J01|Chef|26|Michigan|
|E002|Bob|J02|Waiter|66|California|
|E003|Tom|J02|Waiter|51|Oregon|


### 제 2 정규형(2NF; Second Normal Form)
---
제 2 정규형은 1NF을 만족하면서, Primary Key를 중심으로 의존 결과를 알 수 있어야 한다. 부분적인 의존도가 없어야 하는데, 즉 모든 부가 속성들은 Primary Key를 가지고 찾을 수 있어야 한다. 해당 정규형의 목표는 중복을 제거하고 원자성을 만족하는 것이다.

**2NF 테이블 예제 - Employees & Jobs**

|EMPLOYEE_ID|NAME|STATE_CODE|HOME_STATE|
|---|---|---|---|
|E001|Alice|26|Michigan|
|E002|Bob|56|Wyoming|

|JOB_CODE|JOB|
|---|---|
|J01|Chef|
|J02|Waiter|
|J02|Bartender|

### 제 3 정규형(3NF; Second Normal Form)
---
제 3 정규형은 2NF를 만족하면서, 전이적 부분 종속성이 없어야 한다. 

**3NF 테이블 예제 - Employees & Jobs & States**

|EMPLOYEE_ID|NAME|STATE_CODE|
|---|---|---|
|E001|Alice|26|
|E002|Bob|56|
|E003|Alice|39

|JOB_CODE|JOB|
|---|---|
|J01|Chef|
|J02|Waiter|
|J02|Bartender|

|STATE_CODE|HOME_STATE|
|---|---|
|26|Michigan|
|56|Wyoming|

### SCD(Slowly Changing Dimension)
---
데이터 웨어하우스(DW)나 모든 테이블들의 히스토리를 유지하는 것이 중요하다. 보통 두 개의 timestamp 필드를 갖는 것이 좋다.
- create_at(생성 시간으로 한 번 만들어지면 고정됨)
- update_at(꼭 필요 마지막 수정 시간을 나타냄)

이 경우에 컬럼의 성격에 따라 어떻게 유지할 지 방법이 달라진다.
- SCD Type 0
- SCD Type 1
- SCD Type 2
- SCD Type 3
- SCD Type 4

![SCD](/assets/img/contents/scd.png "Slowly Changing Dimension")

일부 속성들은 시간을 두고 변하게 되는데 DW Table 쪽에 어떻게 반영을 해야 하나? 현재 데이터만을 유지 or 처음부터 지금까지 히스토리도 유지

**SCD Type 0**
해당 타입은 한 번 쓰고 나면 바꿀 이유가 없는 경우에서 사용한다. 관련된 경우로는 한 번 정해지면 갱신되지 않고 고정되는 필드들로, 예시로 고객 테이블의 회원 등록일, 제품 첫 구매일 등이 있다.

**SCD Type 1**
해당 타입은 데이터가 새로 생기면 덮어쓰면 되는 컬럼들의 경우에서 사용된다. 처음 레코드 생성 시에는 존재하지 않았지만 나중에 생기면서 채우는 경우로, 예시로는 고객 테이블의 연간 소득 필드 등이 있다.

**SCD Type 2**
특정 Enity에 대한 데이터가 새로운 레코드로 추가되어야 하는 경우로, 예시로는 고객 테이블의 고객 등급 등이 있다. tier라는 컬럼의 값이 "Regular"에서 "vip"로 변화할 때 변경 시간도 같이 기록되어야 한다.

|customer_id|tier|
|---|---|
|100|vip|
|101|regular|

**SCD Type 3**
SCD Type 2의 대안으로 특정 Entity 데이터가 새로운 컬럼으로 추가되는 경우에 사용하는 타입으로, Type2와 다른 점이라면 이전의 tier 컬럼의 값을 기억하기 위한 새로운 컬럼 previous_tier라는 컬럼을 생성한다. 변경 시간 또한 별도 컬럼으로 존재해야 한다.

|customer_id|tier|previous_tier|
|---|---|---|
|100|vip|regular|
|101|regular|regular|

**SCD Type 4**
특정 Entity에 대한 데이터를 새로운 Dimension 테이블에 저정하는 경우로, 별도의 테이블로 저장하고 이 경우에는 일반화할 수도 있다. 

## dbt(Data Build Tool) 소개
---
`dbt`란 Data Build Tool의 약어로, ELT를 수행하기 위한 오픈소스이다. 데이터 웨어하우스 내에 존재하는 데이터를 변환(In-warehouse data transformation)하기 위해서 사용되며, 해당 tool은 Analytics Engineer라는 말을 만들어 내기도 하였다.

앞서 말했듯이 데이터 웨어하우스와 긴밀한 관계에 있는 만큼, Redshift, Snowflake, Bigquery, Spark 등의 다양한 웨어하우스를 지원한다.

dbt Cloud라는 클라우드 버전도 존재한다.

**dbt를 구성하는 컴포넌트**

![DBT](/assets/img/contents/dbtcomponent.png "DBT Component")

dbt는 **데이터 웨어하우스 환경에서 SQL 기반 데이터 변환을 구조화하기 위한 도구**로, 데이터 모델링과 품질 관리 기능을 함께 제공한다. 주요 구성 요소는 `데이터 모델(models)`, `테스트(tests)`, `스냅샷(snapshots)`이다.

### 데이터 모델 (Models)
---
dbt의 데이터 모델은 SELECT 문으로 정의되며, 실행 결과는 Table 또는 View로 물리화된다. 모델은 데이터 처리 단계에 따라 여러 개의 티어로 구분해 관리하는 것이 일반적이다. 대표적인 티어 구조는 다음과 같다:

**Bronze / Raw Table**
소스 시스템에서 수집한 원본 데이터를 그대로 적재한 레이어이다. 최소한의 가공만 수행하며, 데이터 정합성 이슈 발생 시 재처리의 기준점 역할을 한다.

**Staging Table**
Raw 데이터를 분석에 적합한 형태로 정제하는 단계이다. 컬럼명 표준화, 타입 변환, 기본적인 필터링과 같은 전처리 로직이 적용된다.

**Core Table**
비즈니스 로직이 반영된 핵심 데이터 레이어로, 여러 Staging 테이블을 조합하거나 계산 로직을 적용해 분석 및 리포팅에 바로 사용할 수 있는 형태로 구성된다.

이러한 티어 분리는 데이터 흐름을 명확히 하고, dbt의 Lineage 기능을 통해 모델 간 의존 관계를 체계적으로 관리할 수 있게 한다.

### 데이터 품질 검증 (Tests)
---
dbt는 모델 단위로 데이터 품질 테스트를 정의할 수 있다. 특정 컬럼의 NULL 여부, 유일성, 참조 무결성 등을 검증함으로써 데이터 모델의 신뢰도를 지속적으로 확보할 수 있다.

### 스냅샷 (Snapshots)
---
스냅샷은 테이블의 변경 이력을 관리하기 위한 기능이다. 값이 변경되는 레코드를 시간 축 기준으로 저장함으로써, 과거 상태 기반 분석이나 변경 추적이 가능하다.

## dbt 사용 시나리오
---
데이터 파이프라인에서 요구되는 핵심조건을 만족하기 위해선, 단순히 데이터를 적재하는 것만으로는 불충분하다. 

우선, **데이터 변경 사항을 쉽게 이해할 수 있어야 하며**, 필요할 경우 이전 상태로의 롤백이 가능해야 한다. 이는 데이터 변환 로직과 변경 이력이 명확하게 관리되어야 함을 의미한다.

또한 **데이터 간 리니지(Lineage)를 확인**할 수 있어야 한다. 특정 테이블이나 컬럼이 어떤 소스 데이터와 변환 과정을 거쳐 생성되었는지 추적할 수 있어야, 변경 영향도를 빠르게 파악할 수 있다.

**데이터 품질 테스트와 에러 보고** 역시 필수 요소이다. 데이터 누락, 중복, 무결성 오류 등을 사전에 검증하고, 문제가 발생했을 때 이를 명확하게 인지할 수 있어야 한다.

대용량 데이터 환경에서는 **Fact 테이블의 증분 로드(Incremental Update)**가 중요하다. 전체 데이터를 매번 재처리하지 않고 변경된 데이터만 반영함으로써, 성능과 비용을 효율적으로 관리할 수 있다.

반면, **Dimension 테이블은 변경 이력을 추적**할 수 있어야 한다. 값이 변경되는 시점을 기록하는 히스토리 테이블을 통해, 과거 기준의 분석과 이력 관리가 가능해진다.

마지막으로, 데이터 구조와 변환 로직에 대한 **문서화가 용이**해야 한다. 문서는 단순한 부가 기능이 아니라, 협업과 유지보수를 위한 핵심 요소이며, 코드 기반으로 자동 생성될 수 있을수록 운영 효율이 높아진다.

### Fact 테이블과 Dimension 테이블
---
데이터 웨어하우스 모델링에서 Fact 테이블과 Dimension 테이블은 분석 구조의 핵심을 이룬다. 두 테이블은 역할과 성격이 명확히 구분되며, 함께 사용할 때 분석 효율을 극대화할 수 있다.

**Fact 테이블**
Fact 테이블은 분석의 중심이 되는 **정량적 지표를 저장하는 테이블**이다. 일반적으로 매출, 수익, 판매량 이익과 같은 측정 가능한 값들이 포함되며, 비즈니스 의사결정에 직접적으로 활용된다.

Fact 테이블은 여러 Dimension 테이블과 **외래키(Foreign Key)**를 통해 연결되며, 이를 통해 다양한 관점에서 데이터를 분석할 수 있다. 특성상 이벤트 단위의 데이터가 누적되기 때문에, **테이블 크기가 상대적으로 매우 큰 경우가 많다.**

**Dimension 테이블**
Dimesion 테이블은 Fact 테이블에 대한 **맥락(Context)을 제공하는 테이블**이다. 고객, 제품, 지역과 같은 정보가 여기에 해당하며, Fact 데이터가 어떤 대상과 조건에서 발생했는지를 설명해준다.

각 Dimension 테이블은 Primary Key를 가지며, 이 키는 Fact 테이블에서 Foreign Key로 참조된다. Dimension 데이터를 기준으로 Fact 데이터를 그룹핑하거나 필터링함으로써, 다양한 형태의 분석이 가능해진다. 일반적으로 Dimension 테이블은 Fact 테이블에 비해 데이터 크기가 훨씬 작다.

### ELT 작업 예제
---
AWS Redshift의 데이터 웨어하우스를 사용하여 AB 테스트 분석을 쉽게 진행하기 위한 ELT 테이블을 만든다.
- 입력 테이블: user_event, user_variant, user_metadata
- 출력 테이블: Variant별 사용자별 일별 요약 테이블
    - variant_id, user_id, datastamp, age, gender
    - 총 imporession, 총 clink, 총 purchase, 총 revenue

**입력 데이터들**
Production DB에 저장되는 정보들을 사용할 Redshift에 적재했다고 가정한다.
- raw_data.user_event: 사용자/날짜/아이템별로 impression이 있는 경우 그 정보를 기록하고
impression으로부터 클릭, 구매, 구매시 금액을 기록. 실제 환경에서는 이런
aggregate 정보를 로그 파일등의 소스
- raw_data.user_variant: 사용자가 소속한 AB test variant를 기록한 파일 (control vs. test)
- raw_data.user_metadata: 사용자에 관한 메타 정보가 기록된 파일 (성별, 나이 등등)

```sql
{% raw %}
CREATE TABLE raw_data.user_event(
    user_id int,
    datastamp timestamp,
    item_id int,
    clicked int,
    purchased int,
    paidamount int
);

CREATE TABLE raw_data.user_variant (
user_id int,
variant_id varchar(32) -- control vs. test
);

CREATE TABLE raw_data.user_metadata (
user_id int,
age varchar(16),
);
{% endraw %}
```

**최종 생성 데이터(ELT 테이블)**
SELECT로 표현한 ELT 테이블은 다음과 같다:

```sql
{% raw %}
SELECT
    variant_id,
    ue.user_id,
    datestamp,
    age,
    gender,
    COUNT(DISTINCT item_id) num_of_items, -- 총 impression
    COUNT(DISTINCT CASE WHEN clicked THEN item_id END) num_of_clicks, -- 총 click
    SUM(purchased) num_of_purchases, -- 총 purchase
    SUM(paidamount) revenue -- 총 revenue
FROM raw_data.user_event ue
JOIN raw_data.user_variant uv ON ue.user_id = uv.user_id
JOIN raw_data.user_metadata um ON uv.user_id = um.user_id
GROUP by 1, 2, 3, 4, 5;
{% endraw %}
```

## dbt Models: Input
---
### Model
---
우선 `Model`이란 ELT 기반 데이터 파이프라인에서 **테이블을 생성하기 위한 가장 기본적인 빌딩 블록**이다. dbt에서 Model은 SQL로 정의되며, 실행 결과는 테이블, 뷰, 혹은 CTE 형태로 물리화된다.

Model은 데이터 흐름에 따라 **입력, 중간, 최종 테이블을 정의하는 역할**을 하며, 일반적으로 티어 구조로 관리된다. 예를 들어 `raw → staging → core` 와 같은 단계적 구조를 통해 원본 데이터에서 분석용 데이터까지 점진적으로 변환한다. 이러한 구조는 데이터 처리 책임을 명확히 하고, 변환 로직의 가독성과 유지보수성을 높인다.

**Model의 구성 요소**
dbt의 Model은 입력 데이터로부터 최종 결과 테이블까지의 변환 과정을 정의하는 단위이며, 크게 **Input과 Output으로 구분**할 수 있다.

`Input`
Input은 변환의 시작점이 되는 데이터로, **raw 및 staging(src) 단계의 데이터**를 의미한다. raw 데이터는 일반적으로 **CTE 형태로 정의**되며, 소스 시스템에서 적재된 원본 데이터를 그대로 참조한다. staging 단계는 raw 데이터를 정제하고 구조를 표준화하는 역할을 하며, **보통 View로 물리화**된다.

`Output`
Output은 분석에 직접 사용되는 **최종(core) 데이터**를 의미한다. core 모델은 비즈니스 로직이 적용된 결과물로, 일반적으로 **Table 형태로 생성**된다. 이 단계의 데이터는 Fact 및 Dimension 테이블의 기반이 된다.


**Model 관리 방식**
이러한 모든 Model은 **models 디렉토리 하위에 SQL 파일 형태로 관리**된다. 각 SQL 파일은 기본적으로 SELECT 문으로 구성되며, dbt의 `Jinja 템플릿과 매크로`를 함께 사용해 재사용성과 가독성을 높일 수 있다.

또한 Model 내에서는 다른 테이블이나 Model을 직접 참조(reference)할 수 있으며, 이를 통해 dbt는 **모델 간 의존 관계와 리니지를 자동으로 추적**한다. 이는 데이터 변경 시 영향 범위를 빠르게 파악하는 데 중요한 역할을 한다.

### View
---
여기서 잠깐 `View`란 SELECT 쿼리의 결과를 기반으로 생성되는 **가상의 테이블**이다. 하나의 테이블 일부 데이터만 노출하거나, 여러 테이블을 조인한 결과를 하나의 논리적 테이블처럼 제공할 수 있다. 일반적으로 `CREATE VIEW view_name AS SELECT ...` 형태로 정의된다.

View를 사용함으로써 얻을 수 있는 **장점**으로는 데이터 접근에 대한 **추상화 계층**을 제공한다는 점이다. 사용자는 View를 통해 필요한 데이터에만 접근하면 되며, 원본 테이블의 구조나 복잡한 조인 로직을 알 필요가 없다. 또한 View를 활용하면 **사용자에게 필요한 데이터만 노출할 수 있어 보안 측면에서도 유리**하다. 반복적으로 사용되는 복잡한 쿼리를 View로 정의함으로써, 쿼리 자체를 단순화하는 효과도 있다.

반면, View는 쿼리가 실행될 때마다 **원본 데이터를 조회하기 때문에 성능 저하가 발생**할 수 있다는 **단점**이 있다. 또한 원본 테이블 구조가 변경되었음을 인지하지 못한 상태에서 View를 실행하면 오류가 발생할 수 있다는 점도 고려해야 한다.

`CTE(Common Table Expression)`
CTE는 SQL 쿼리 내에서 임시 결과 집합을 정의하고 재사용하기 위한 구조이다. `WITH`절을 사용하여 하나 이상의 서브 쿼리를 정의한 뒤, 이후 메인 쿼리에서 이를 참조할 수 있다.

일반적인 CTE 구조는 다음과 같다. 여러 개의 CTE를 정의할 수 있으며, 각 CTE는 논리적으로 분리된 변환 단계를 표현한다:

```sql
{% raw %}
WITH temp1 AS (
    SELECT k1, k2
    FROM t1
    JOIN t2 ON t1.id = t2.foreign_id
),
temp2 AS (
    ...
)
SELECT *
FROM temp1 t1
JOIN temp2 t2 ON ...
{% endraw %}
```

CTE를 사용하면 **복잡한 쿼리를 단계별로 나누어 작성**할 수 있어 **가독성과 유지보수성이 크게 향상**된다.

**dbt에서 CTE 활용**
dbt에서는 CTE를 활용해 **raw 데이터를 논리적으로 분리된 입력 소스(src)**로 정의하는 패턴을 자주 사용한다. 예를 들어 원본 테이블을 직접 참조하는 대신, CTE로 한 번 감싸서 이후 변환 로직의 기준점으로 삼는다.

```sql
{% raw %}
-- models/src - src_user_event.sql
WITH src_user_event AS (
    SELECT *
    FROM raw_data.user_event
)
SELECT
    user_id,
    datestamp,
    item_id,
    clicked,
    purchased,
    paidamount
FROM src_user_event

-- models/src - src_user_variant.sql
WITH src_user_variant AS (
    SELECT * FROM raw_data.user_variant
) 
SELECT
    user_id,
    variant_id
FROM
    src_user_variant

-- models/src - src_user_metadata.sql
WITH src_user_metadata AS (
    SELECT * FROM raw_data.user_metadata
) 
SELECT
    user_id,
    age,
    gender,
    updated_at
FROM
    src_user_metadata
{% endraw %}
```

**Model 빌딩**

```shell
dbt run
```

이와 같은 방식은 원본 데이터 접근을 명확히 하고, 이후 컬럼 선택이나 변환 로직을 단계적으로 확장하기에 용이하다. 특히 dbt 모델 내에서 CTE는 **Input 레이어를 구성하는 핵심 요소로 활용**된다.

## dbt  Models: Output
---
### Materialization
---
`Materialization`은 입력 데이터(테이블 또는 모델)를 기반으로 **새로운 데이터 모델을 실제로 어떻게 생성할 것인지를 정의하는 방식**이다. dbt에서는 여러 입력 모델을 연결해 하나의 결과를 만들며, 이 과정에서 추가적인 변환 로직이나 데이터 클린업이 수행된다.

Materialization은 모델 단위로 설정할 수 있으며, 파일 레벨 또는 프로젝트 레벨에서 지정 가능하다. 설정된 Materialization 방식은 `dbt run` 실행 시 적용되며, 실행 옵션에 따라 동작을 제어할 수도 있다.

**dbt에서 제공하는 Materialization 유형**
dbt에서는 기본적으로 네 가지의 Materialization 방식을 제공한다:

- `View`: 데이터를 자주 사용하지 않거나, 항상 최신 상태를 유지해야 하는 경우에 적합하다. 저장 공간을 거의 사용하지 않지만, 조회 시마다 원본 쿼리가 실행된다.
- `Table`: 데이터를 반복적으로 자주 사용하는 경우에 적합하다. 실행 시 결과를 물리적인 테이블로 생성하므로, 조회 성능이 안정적이다.
- `Incremental`: 기존 테이블에 새로운 데이터만 추가하는 방식으로, 주로 **Fact 테이블에 사용**된다. 과거 레코드를 수정할 필요가 없는 경우에 적합하며, 대용량 데이터 처리 시 성능과 비용 측면에서 효과적이다.
- `Ephemeral`: 물리적인 테이블이나 뷰를 생성하지 않고, **CTE 형태로 쿼리 내에 인라인**된다. 하나의 SELECT 문에서 반복적으로 사용되는 로직을 모듈화하는 데 주로 활용된다.

### Jinja 템플릿
---
Jinja는 Python 기반의 템플릿 엔진으로, Flask와 같은 웹 프레임워크에서 널리 사용되며 Airflow에서도 활용된다. dbt에서는 Jinja를 활용하여 SQL을 동적으로 생성한다.

Jinja 템플릿은 입력 파라미터를 기반으로 SQL을 유연하게 구성할 수 있으며, 조건문, 반복문, 필터와 같은 기능을 제공한다. 이를 통해 중복되는 SQL 로직을 줄이고, 유지보수성을 높일 수 있다.

### Core 모델 디렉터리 구성
---
dbt 프로젝트에서는 분석에 직접 사용되는 **core 테이블을 별도의 디렉토리로 분리**해 관리하는 것이 일반적이다. 이를 통해 Fact와 Dimension 모델의 역할을 명확히 구분할 수 있다.

`models` 디렉토리 하위에는 core 레이어를 위한 폴더를 생성하고, 그 아래에 **`dim`과 `fact` 폴더를 각각 구성**한다.

- `dim` 폴더에는 Dimension 테이블에 해당하는 모델을 정의한다. 예를 들어 사용자 관련 정보를 담는 `dim_user_variant.sql`, `dim_user_metadata.sql`과 같은 파일을 생성한다.

- `fact` 폴더에는 Fact 테이블에 해당하는 모델을 정의한다. 사용자 이벤트 데이터를 저장하는 `fact_user_event.sql`과 같이 분석의 중심이 되는 지표 데이터를 관리한다.

이렇게 정의된 core 모델들은 모두 **물리적인 Table 형태로 생성**되며, 최종적으로 리포팅과 분석에 사용된다. 디렉토리 구조를 통해 모델의 성격을 명확히 구분함으로써, 프로젝트 전반의 가독성과 유지보수성을 높일 수 있다.

```sql
{% raw %}
-- models/dim - dim_user_variant.sql
WITH src_user_variant AS (
    SELECT * FROM {{ ref('src_user_variant')}}
)
SELECT
    user_id,
    variant_id
FROM
    src_user_variant

-- models/dim - dim_user_metadata.sql
WITH src_user_metadata AS (
    SELECT * FROM {{ ref('src_user_metadata') }}
) 
SELECT
    user_id,
    age,
    gender,
    updated_at
FROM
    src_user_metadata

-- models/fact - fact_user_event.sql
{{
    config(
        materialized = 'incremental',
        on_schema_change='fail'
    )
}}
WITH src_user_event AS (
    SELECT * FROM {{ ref("src_user_event") }}
)
SELECT
    user_id,
    datastamp,
    item_id,
    clicked,
    purchased,
    paidamount
FROM
    src_user_event
{% endraw %}
```

**Model 빌딩**

```shell
dbt run
```

**ddbt compile vs dbt run**
dbt compile은 SQL 코드까지만 생성하고 실행하지 않지만, dbt run은 생성된 코드를 실제 실행에 옮긴다.


## dbt Seeds
---
`dbt Seeds`는 데이터 웨어하우스에서 사용되는 Dimension 테이블 중에는 **데이터 크기가 작고 변경이 거의 없는 경우**가 많다. dbt의 Seeds 기능은 이러한 **데이터를 파일 형태로 웨어하우스에 로드하기 위한 방법**이다.

Seeds는 보통 **CSV 파일**로 관리되며, 코드와 함께 버전 관리가 가능하다. `dbt seed` 명령을 실행하면 해당 파일이 테이블로 생성되어, 소규모 기준 데이터나 고정된 매핑 테이블을 관리하는 데 유용하다.

**입력 테이블 변경과 Sources의 필요성**
Staging 테이블을 생성할 때 입력 테이블 구조나 이름이 자주 변경된다면, `models` 디렉토리 하위의 SQL 파일을 일일이 수정해야 하는 문제가 발생한다. 이러한 번거로움을 해결하기 위해 **dbt는 Sources 개념을 제공**한다.

Sources는 ETL 단계에서 최초로 유입되는 테이블을 대상으로 하며, **입력 테이블에 별칭(alias)을 부여**하고 이를 staging 모델에서 참조하도록 한다. 이를 통해 소스 테이블명이 변경되더라도 downstream 모델에는 영향을 최소화할 수 있다.



## dbt Sources
---
**Sources 개념**
Sources는 원본 테이블에 대한 추상화 계층이다. 실제 테이블 이름 대신 source 이름과 테이블 이름의 조합으로 참조하며, 예를 들어 `raw_data.user_metadata` 테이블을 `username.metadata`와 같은 형태로 정의할 수 있다.

이 방식은 ETL 단의 변경 사항을 효과적으로 캡슐화하고, 모델 간 의존성을 낮춰 유지보수를 용이하게 한다. 또한 Sources는 단순한 별칭 제공을 넘어, **소스 테이블에 새로운 레코드가 존재하는지 여부를 체크하는 기능도 함께 제공**한다.

**Sources 최신성 (Freshness)**
dbt는 소스 데이터의 **최신성(Freshness)**을 확인할 수 있는 기능을 제공한다. 이는 특정 소스 테이블의 데이터가 기준 시점 대비 얼마나 오래되었는지를 검증하는 기능이다.

`dbt source freshness` 명령을 통해 실행할 수 있으며, 이를 위해 `models/sources.yml` 파일에서 각 Source 테이블에 대한 최신성 기준을 정의해야 한다. 이 기능을 활용하면 ETL 지연이나 데이터 수집 문제를 조기에 감지할 수 있다.

```yaml
version: 2

sources:
    - name: username
      schema: raw_data
      tables:
        - name: metadata
          identifier: user_metadata
        - name: event
          identifier: user_event
        - name: variant
          identifier: user_variant
```

```sql
{% raw %}
WITH src_user_event AS (
    SELECT * FROM raw_data.user_event
)
SELECT
    user_id,
    datastamp,
    item_id, 
    clicked, 
    purchased,
    paidamount
FROM
    src_user_event
{% endraw %}
```

## dbt Snapshots
---
데이터 웨어하우스에서 Dimension 테이블은 **특성상 값 변경이 자주 발생**할 수 있다. dbt에서의 `스냅샷`은 이러한 테이블의 변화를 지속적으로 기록함으로써, **과거 특정 시점의 데이터 상태를 다시 조회할 수 있도록 하는 기능**을 의미한다.

스냅샷을 활용하면 데이터에 문제가 발생했을 경우 과거 데이터 기준으로 롤백이 가능하며, 값 변경 이력을 기반으로 다양한 데이터 이슈를 보다 쉽게 디버깅할 수 있다.

**SCD Type 2와 dbt**
SCD(Slowly Changing Dimension) Type 2는 Dimension 테이블에서 특정 엔티티의 값이 변경될 때, **기존 레코드를 유지한 채 새로운 레코드를 추가하는 방식**이다.

예를 들어 `employee_jobs` 테이블에서 특정 `employee_id의` `job_code`가 변경되는 경우, 기존 데이터는 종료 시점을 기록하고 새로운 데이터에는 변경된 값과 함께 변경 시점을 추가한다. 이를 통해 시간에 따른 상태 변화를 추적할 수 있다.

dbt에서는 이러한 SCD Type 2 패턴을 **스냅샷 테이블**을 통해 구현한다. 변경이 발생할 때마다 새로운 레코드를 생성하는 별도의 히스토리 테이블을 유지함으로써, 과거 상태 분석이 가능해진다.

**dbt의 스냅샷 처리 방식**
dbt에서 스냅샷은 `snapshots` 디렉토리 하위에 정의된다. 스냅샷을 적용하기 위해서는 대상 데이터 소스가 몇 가지 조건을 만족해야 한다.

우선, **Primary Key가 반드시 존재**해야 하며, 레코드 변경 시점을 판단할 수 있는 **타임스탬프 컬럼(updated_at, modified_at 등)**이 필요하다. dbt는 Primary Key를 기준으로 변경 시간을 비교해, 현재 데이터 웨어하우스에 저장된 시점보다 최신인 경우 변경으로 판단한다.

스냅샷 테이블에는 변경 이력을 관리하기 위해 총 네 개의 주요 컬럼이 생성된다.
`dbt_scd_id`와 `dbt_updated_at`은 dbt 내부 관리용 컬럼이며, `valid_from`과 `valid_to`는 해당 레코드가 유효한 기간을 나타낸다.

```sql
{% raw %}
{% snapshot scd_user_metadata %}

{{
    config(
    target_schema='keeyong',
    unique_key='user_id',
    strategy='timestamp',
    updated_at='updated_at',
    invalidate_hard_deletes=True
)
}}

SELECT * FROM {{ source('keeyong', 'metadata') }}

{% endsnapshot %}
{% endraw %}
```

**dbt snapshot 실행**

```shell
dbt snapshot
```

## dbt Tests
---
dbt는 데이터 변환 과정에서 **데이터 품질을 검증하기 위한 테스트 기능을 제공**한다. 테스트는 모델이 기대한 규칙을 만족하는지 확인하는 역할을 하며, 데이터 파이프라인의 신뢰성을 유지하는 핵심 요소이다.

dbt의 테스트는 크게 **Generic 테스트**와 **Singular 테스트** 두 가지로 구분된다.

`Generic Tests`
Generic 테스트는 **dbt에서 기본적으로 제공하는 내장 테스트**로, 자주 사용되는 데이터 품질 검증 패턴을 손쉽게 적용할 수 있다. 대표적으로 `unique`, `not_null`, `accepted_values`, `relationships`와 같은 테스트를 지원한다.

이러한 테스트는 주로 `models` 디렉토리 내의 YAML 파일에서 정의되며, 컬럼 단위로 간단하게 설정할 수 있다. 반복적인 품질 검증 로직을 표준화하는 데 적합하다.

`Singular Tests`
Singular 테스트는 사용자가 직접 정의하는 `커스텀 테스트`이다. 기본적으로 하나의 `SELECT` 쿼리로 작성되며, 쿼리 실행 결과가 `한 행이라도 반환되면 테스트는 실패`로 간주된다.

Singular 테스트는 `tests` 디렉토리 하위에 SQL 파일로 관리되며, 복잡한 비즈니스 규칙이나 Generic 테스트로 표현하기 어려운 검증 로직을 구현할 때 활용된다.

## dbt Documentations
---
dbt의 문서화 기능은 문서와 소스 코드를 최대한 가깝게 배치한다는 철학을 기반으로 한다. 데이터 모델과 그에 대한 설명을 분리하지 않고 함께 관리함으로써, 코드 변경과 문서 갱신이 자연스럽게 동기화되도록 한다.

dbt에서 문서를 작성하는 방법은 크게 두 가지가 있다. 첫 번째는 기존의 YAML 파일에 문서 내용을 함께 작성하는 방식으로, 모델과 컬럼 정의 옆에 설명을 추가할 수 있다. 이 방식은 코드와 문서의 일관성을 유지하기 쉬워 일반적으로 선호된다. 두 번째 독립적인 Markdown 파일을 생성하는 방식으로, 보다 자유로운 형태의 설명이나 개념 정리에 적합하다.

작성된 문서는 dbt에서 제공하는 경량 웹 서버를 통해 서빙할 수 있다. 이때 `overview.md` 파일은 문서 사이트의 기본 홈 페이지 역할을 하며, 이미지와 같은 정적 자산도 함께 포함할 수 있다. 이를 통해 dbt 프로젝트 전반에 대한 구조와 맥락을 한눈에 파악할 수 있는 문서 환경을 구축할 수 있다.

```yaml
version: 2 

models: 
    - name: dim_user_metadata
      description: A dimension table with user metadata
      columns:
      - name: user_id
        description: The Primary key of the table
        tests:
        - unique
        - not null
```

**models 문서 만들기**
사용자 권한이 있다면 Redshift로부터 더 많은 정보를 가져다가 보여준다. 결과 파일은 target/catalog.json 파일로 출력된다.

```shell
dbt docs generate
```

## dbt Expectations
---
dbt Expectations는 **Greate Expectations에서 영감을 받아 개발된 dbt용 확장 패키지**로, dbt의 기본 테스트 기능을 보다 풍부하게 확장해준다. dbt 환경에서 **데이터 품질 규칙을 선언적으로 정의할 수 있도록 돕는 라이브러리**이다.

패키지는 Github에서 제공되며, 설치 후 `packages.yml` 파일에 등록하여 사용한다. 특정 버전 범위를 명시함으로써 프로젝트 환경에 맞는 안정적인 테스트 구성이 가능하다.

```yaml
packages:
    - package: calogica/dbt_expectations
      version: [">=0.7.0", "<0.8.0"]
```

dbt Expectations는 일반적으로 dbt에서 기본 제공하는 테스트들과 함께 사용되며, 테스트 정의는 주로 `models/schema.yml` 파일에서 관리된다. 이를 통해 기본 테스트로는 표현하기 어려운 보다 정교한 데이터 품질 규칙을 적용할 수 있다.

**dbt Expectations 주요 함수**
dbt Expectations는 다양한 테스트 함수를 제공한다. 예를 들어 컬럼 존재 여부를 검증하는 `expect_column_to_exist`, 최근 데이터가 정상적으로 유입되고 있는지를 확인하는 `expect_row_values_to_have_recent_data`와 같은 테스트가 있다.

또한 컬럼 값의 NULL 여부, 유일성, 데이터 타입, 값의 범위나 허용 집합 여부 등을 검증하는 함수들도 제공된다. 이러한 함수들을 활용하면 데이터 품질 규칙을 코드로 명확히 정의하고, 운영 환경에서 일관되게 검증할 수 있다.

## (번외) 데이터 카탈로그
---
데이터 카탈로그는 조직 내에 존재하는 **데이터 자산의 메타데이터를 중앙에서 관리하는 저장소**이다. 이는 데이터 거버넌스를 위한 출발점으로, 많은 기업에서는 데이터 카탈로그 자체를 거버넌스 도구로 활용하거나, 이를 기반으로 커스텀 기능을 확장해 사용한다.

데이터 카탈로그의 핵심 기능 중 하나는 **메타데이터의 (반)자동 수집**이다. 이를 통해 데이터 자산의 구조, 사용 현황, 연관 관계를 효율적으로 관리할 수 있다. 또한 일반적으로 **실제 데이터가 아닌 메타데이터만 접근**하기 때문에, 데이터 보안 측면에서도 중요한 역할을 한다.

### 데이터 자산의 종류
---
데이터 카탈로그에서 관리되는 자산은 데이터베이스 테이블에 국한되지 않는다. 분석에 활용되는 **대시보드, 문서 및 메시지 도구(Slack, Jira, GitHub 등), 머신러닝 피처, 데이터 파이프라인, 그리고 사용자 정보(HR 시스템)**까지 조직 내 다양한 데이터 관련 자산이 관리 대상이 된다.

### 데이터 카탈로그의 역할
---
데이터 카탈로그는 데이터 자산을 **다양한 관점에서 조직적으로 관리하기 위한 프레임워크**이다. 비즈니스 용어와 데이터 용어를 구분해 **관리하거나, 태그 기반 분류를 통해 데이터 탐색성을 높일 수 있다.**

또한 각 데이터 자산에는 **비즈니스 오너와 기술 오너**를 명확히 지정해 책임 소재를 분명히 하며, 표준화된 문서 템플릿을 통해 데이터 이해도를 높이고 협업을 용이하게 한다.

### 데이터 카탈로그의 주요 기능
---
데이터 카탈로그는 단순한 메타데이터 저장소를 넘어, **조직 전반의 데이터 활용과 거버넌스를 지원하는 핵심 플랫폼이다. 이를 위해 다양한 기능을 제공**한다.

우선 주요 데이터 플랫폼과의 폭넓은 연동 지원이 필수적이다. 데이터 웨어하우스, BI 도구, ETL/ELT 도구, 오케스트레이션 도구 등 여러 시스템에서 메타데이터를 수집해 하나의 통합된 뷰로 제공한다.

또한 **비즈니스 용어집(Business Glossary)**을 통해 비즈니스 관점의 용어와 실제 데이터 자산을 연결하고, 주석·문서·태그와 같은 협업 기능을 제공함으로써 데이터에 대한 조직 내 공통 이해를 형성한다. 여기에 데이터 리니지, 모니터링, 감사 및 트레이싱 기능을 통해 데이터의 흐름과 사용 이력을 추적할 수 있다.

강력한 검색 기능 역시 중요한 요소이다. 단순 키워드 검색을 넘어 통합 검색이나 NLP 기반 검색을 지원하며, 사용자 역할(예: 마케팅 분석가)별로 데이터 추천을 제공해 데이터 탐색 효율을 높인다.

### 주요 데이터 플랫폼 지원
---
데이터 카탈로그는 다양한 데이터 생태계와 연동된다. Redshift, Snowflake, BigQuery와 같은 **데이터 웨어하우스 및 데이터 레이크**, Looker, Tableau, Power BI 등 BI 도구, dbt, Spark, Hive와 같은 ELT 처리 도구, Airflow와 같은 **ETL 오케스트레이션 도구**가 대표적이다.

이 외에도 Cassandra, Druid, Elasticsearch, Kafka Schema Registry와 같은 NoSQL 시스템이나 CSV 파일, 그리고 Azure AD, LDAP과 같은 사용자 관리 시스템까지 연계 대상에 포함된다.

### 데이터 리니지 기능
---
데이터 카탈로그의 핵심 기능 중 하나는 **데이터 리니지(Lineage)**이다. 리니지는 데이터가 어디서 생성되어 어떤 과정을 거쳐 소비되는지를 시각적으로 보여준다.

리니지는 데이터셋 간(dataset-to-dataset) 관계를 SQL 파싱을 통해 추적할 수 있으며, `입력 데이터셋 → 파이프라인 → 출력 데이터셋`으로 이어지는 **파이프라인 단위의 흐름**도 관리한다. 또한 하나의 차트가 여러 대시보드에서 사용되는 경우를 고려한 dashboard-to-chart, chart-to-dataset 리니지도 중요하다.
특히 dbt와 같은 도구는 모델 간 의존성이 명확해, job-to-dataflow 관점에서 정교한 리니지 구성이 가능하다.

### 데이터 거버넌스 관점에서의 중요성
---
데이터 카탈로그는 조직이 보유한 **모든 데이터 자산에 대한 통합 뷰를 제공**한다. 이를 통해 데이터 탐색과 이해에 소요되는 시간이 줄어들고, 설문이나 데이터 티켓 감소로 이어져 전반적인 생산성이 향상된다.

또한 **잘못된 데이터 사용이나 개인정보 확산과 같은 리스크를 줄이고, 불필요한 데이터 생성이나 사용되지 않는 데이터셋을 제거**함으로써 **인프라 비용 절감 효과**도 기대할 수 있다. 컬럼 레벨 리니지와 CI/CD 프로세스를 연계하면, 데이터 변경으로 인한 장애와 이슈 역시 효과적으로 감소시킬 수 있다.

### 데이터 카탈로그 이후의 다음 단계
---
데이터 카탈로그 구축 이후에는 **자동화된 데이터 거버넌스 워크플로우**를 점진적으로 추가하는 것이 다음 단계가 된다. 그 시작점으로 데이터 품질 관련 경보 시스템을 구현할 수 있다.

예를 들어 중요한 메타데이터 변경이나 데이터 품질 이슈 발생 시 알림을 제공하거나, 관심 있는 데이터 자산의 오너나 정의가 변경될 경우 자동으로 통지하는 방식이다. 더 나아가 데이터 관련 지표를 정기적으로 리뷰하는 미팅을 운영함으로써, 데이터 거버넌스를 조직 문화로 정착시킬 수 있다.