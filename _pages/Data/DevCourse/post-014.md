---
title: "[6기] 데브코스 DE WIL 14 | 대용량 데이터 처리 Spark & SparkML"
tags:
    - DevCourse
    - Data Engineering
    - SparkML
date: "2025-06-30"
thumbnail: "/assets/img/thumbnail/devcourse.png"
bookmark: true
---

## 이번 주 학습 목표 
--- 
- 
- 
-

## Spark 기타 기능과 메모리 관리
---
대규모 데이터 처리 환경에서 성능 병목의 가장 흔한 원인 중 하나는 **셔플(Shuffle)**이다. Spark에서는 이를 줄이기 위한 다양한 최적화 기법을 제공하며, 그중 대표적인 방식이 **Broadcast Variable**이다. Broadcast Variable은 특히 머신러닝 파이프라인이나 룩업 테이블 처리에서 매우 중요한 역할을 한다.

### Broadcast Variable란
---
Broadcast Variable은 작은 크기의 데이터를 모든 Executor에 미리 전달하여 공유하는 방식이다. 이를 통해 각 태스크가 해당 데이터를 직접 가져오기 위해 셔플을 발생시키는 상황을 방지할 수 있다.

이 방식은 브로드캐스트 조인(Broadcast Join)에서 사용되는 기법과 동일한 원리이며, 보통 룩업 테이블이나 디멘션 테이블을 다룰 때 사용된다. 많은 데이터 웨어하우스 환경에서는 스타 스키마 형태로 팩트 테이블과 디멘션 테이블이 분리되어 있는데, 디멘션 테이블은 크기가 상대적으로 작기 때문에 브로드캐스트에 적합하다. 일반적으로 10~20MB 정도의 데이터가 그 기준이 된다.

Spark에서는 `spark.sparkContext.broadcast`를 사용해 Broadcast Variable을 생성한다.

**Closure 방식과 Broadcast 방식의 차이**
Spark에서 UDF 내부에서 외부 데이터를 사용하는 방식은 크게 두 가지로 나뉜다. 하나는 Closure를 사용하는 방식이고, 다른 하나는 Broadcast Variable을 사용하는 방식이다.

`Closure` 방식에서는 파이썬 데이터 구조가 태스크 단위로 직렬화된다. 즉, 각 태스크마다 동일한 데이터가 반복적으로 전송되며, 이는 네트워크와 메모리 측면에서 비효율적이다. UDF 내부에서 일반 파이썬 변수나 컬렉션을 참조할 경우 이 방식이 사용된다.

반면 `Broadcast` 방식에서는 **데이터가 Worker Node 단위로 한 번만 직렬화되어 전달**된다. 이후 해당 데이터는 Executor 내에서 캐싱되며, 여러 태스크가 이를 공유해서 사용한다. 따라서 UDF 안에서 브로드캐스트된 데이터를 참조하는 방식은 훨씬 효율적이다.

Broadcast 데이터셋은 몇 가지 특징을 가진다. Worker Node로 공유되는 데이터는 변경이 불가능하며, 노드별로 한 번만 전송되어 캐싱된다. 단, 이 데이터는 Executor의 Task Memory에 적재되어야 하므로 크기에 제한이 있다.

**Broadcast Variable 활용 예제**
Broadcast Variable의 활용을 이해하기 위해, 간단한 예제를 살펴본다. 특정 코드에 해당하는 이름을 찾아야 하는 상황을 가정한다. 이때 룩업 테이블을 DataFrame으로 로딩한 뒤 조인을 수행할 수도 있지만, 룩업 테이블이 작다면 브로드캐스트하여 UDF 안에서 사용하는 방식이 더 효율적일 수 있다.

아래 예제에서는 CSV 파일로부터 룩업 테이블을 읽어 Map 형태로 변환한 뒤, 이를 Broadcast Variable로 생성한다. 이후 UDF에서 브로드캐스트된 데이터를 참조하여 코드를 이름으로 변환한다.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *

def my_func(code: str) -> str:
    return bdData.value.get(code)

if __name__ == '__main__':
    spark = SparkSession \
        .builder \
        .appName("Demo") \
        .master("local[3]") \
        .getOrCreate()

    prdCode  = spark.read.csv("data/lookup.csv")/rdd.collectAsMap()

    bdData = spark.sparkContext.broadcast(prdCode)

    data_list = [("98312", "2021-01-01", "1200", "01"),
                 ("01056", "2021-01-02", "2345", "01"),
                 ("98312", "2021-02-03", "1200", "02"),
                 ("01056", "2021-02-04", "2345", "02"),
                 ("02845", "2021-02-05", "9812", "02")]
    df = spark.createDataFrame(data_list) \
        .toDF("code", "order_date", "price", "qty")

    spark.udf.register("my_udf", my_func, StringType())
    df.withColumn("Product", expr("my_udf(code)")) \
        .show()
```
해당 방식은 셔플을 발생시키지 않으며, 룩업 테이블을 반복적으로 전송하지 않기 때문에 대규모 데이터 처리 환경에서는 성능상 큰 이점을 가진다.

### 실행 및 리소스 관리(Accumulators & Speculative Execution)
---
Spark로 대규모 데이터를 처리하다 보면, 단순한 변환 로직뿐 아니라 모니터링, 성능 안정화, 리소스 활용 방식까지 함께 고려해야 한다. **Accumulators와 Speculative Execution, 그리고 리소스 할당 방식은 이러한 운영 관점에서 중요한 개념**들이다.

**Accumulators**
Accumulators는 Spark에서 **특정 이벤트의 수나 합계를 기록하기 위한 전역 변수**이다. 개념적으로는 Hadoop의 Counter와 매우 유사하며, 예를 들어 비정상적인 값을 가진 레코드 수를 집계하는 데 자주 사용된다.

Accumulators는 드라이버에 위치한 변경 가능한 전역 변수이며, Executor에서 값을 누적한 뒤 드라이버로 전달된다. 스칼라 타입으로 생성한 경우에만 이름을 지정할 수 있고, 이름이 지정된 Accumulator만 Spark Web UI에서 확인할 수 있다.

**Accumulators 사용 시 주의 사항**
Accumulators는 레코드별 카운트나 합계 계산에 사용할 수 있지만, 어디에서 사용하느냐에 따라 값의 정확도가 달라진다.

Transformation 내부에서 Accumulator를 사용하는 경우, 태스크 재시도나 Speculative Execution으로 인해 값이 중복 반영될 수 있다. 따라서 이 방식에서는 Accumulator 값이 부정확해질 수 있다.

반면 DataFrame이나 RDD의 `foreach`와 같은 액션 단계에서 사용하는 경우에는 정확한 값이 보장된다. 이 방식이 Accumulator 사용 시 권장되는 접근 방식이다.

**Speculative Execution**
Speculative Execution은 **느리게 실행되는 태스크를 다른 Executor에서 중복 실행하는 기능**이다. 특정 Worker Node의 하드웨어 문제나 일시적인 성능 저하로 인해 태스크가 늦어질 경우, 전체 잡의 완료 시간을 줄이기 위한 목적을 가진다.

하지만 태스크 지연의 원인이 데이터 스큐(Data Skew)인 경우에는 도움이 되지 않으며, 오히려 리소스만 낭비하게 될 수 있다. 이 때문에 Speculative Execution은 상황에 따라 신중하게 사용해야 한다.

**Speculative Execution 제어**
Speculative Execution은 `spark.speculation` 옵션으로 제어할 수 있으며, 기본값은 비활성화(false)다. 이 기능은 Hadoop MapReduce 시절부터 존재해왔으며, 다양한 환경 변수를 통해 세밀하게 조정할 수 있다.

예를 들어 태스크 실행 시간 기준, 상위 지연 태스크 비율, 최소 실행 시간 등을 조정함으로써 Speculative Execution의 민감도를 제어할 수 있다. 대규모 클러스터 환경에서는 기본값보다 보수적인 설정을 사용하는 경우도 많다.

### Spark 리소스 할당 방식
---
Spark에서는 두 가지 수준에서 리소스 할당이 이루어진다.

첫 번째는 **Spark Application** 간의 리소스 할당이며, 이는 YARN과 같은 리소스 매니저가 담당한다. YARN은 FIFO, FAIR, CAPACITY와 같은 스케줄링 방식을 지원한다.

두 번째는 **하나의 Spark Application 내부에서 잡(Job) 간 리소스 할당**이다. 기본적으로는 FIFO 방식으로, 먼저 실행된 잡이 필요한 만큼 리소스를 우선 사용한다.

**리소스 요구와 해제 방식**
Spark Application의 리소스 사용 방식에는 두 가지가 있다.

Static Allocation은 기본 동작 방식으로, Spark Application이 리소스 매니저로부터 할당받은 Executor를 애플리케이션 종료 시점까지 유지한다. 이 방식은 단순하지만, 클러스터 전체의 리소스 사용률을 떨어뜨릴 수 있다.

Dynamic Allocation은 실행 상황에 따라 Executor를 요청하거나 반환하는 방식이다. 여러 Spark Application이 하나의 리소스 매니저를 공유하는 환경에서는 Dynamic Allocation을 활성화하는 것이 일반적으로 유리하다.

리소스 설정은 spark-submit 명령어를 통해 `--num-executors`, `--executor-cores`, `--executor-memory`와 같은 옵션으로 제어할 수 있다.

### Spark의 리소스 할당 전략과 스케줄링
---
Spark 애플리케이션의 성능과 클러스터 활용 효율은 **리소스를 어떻게 할당하고 스케줄링하느냐**에 크게 좌우된다. Spark는 실행 환경과 워크로드 특성에 따라 다양한 리소스 관리 전략을 제공하며, 그중 핵심이 Static Allocation과 Dynamic Allocation, 그리고 Spark Scheduler다.

**Static Allocation과 Dynamic Allocation**
Spark의 기본 리소스 할당 방식은 Static Allocation이다. 이 방식에서는 spark-submit 시점에 지정한 Executor 수와 리소스를 애플리케이션 종료 시점까지 유지한다.

```shell
spark-submit —num-executors 100 —executor-cores 4 —executor-memory 32G
```

Static Allocation은 설정이 단순하고 예측 가능하지만, 작업 부하가 줄어들어도 리소스를 반환하지 않기 때문에 클러스터 전체 관점에서는 리소스 낭비로 이어질 수 있다.

이에 비해 Dynamic Allocation은 실행 상황에 따라 Executor를 동적으로 요청하거나 반환하는 방식이다. 작업이 몰릴 때는 Executor를 늘리고, 유휴 상태가 지속되면 Executor를 릴리스함으로써 리소스 사용 효율을 높인다. 여러 Spark Application이 하나의 클러스터를 공유하는 환경에서는 Dynamic Allocation이 특히 효과적이다.

**Dynamic Resource Allocation 제어 옵션**
Dynamic Allocation은 여러 환경 변수를 통해 세밀하게 제어할 수 있다. `spark.dynamicAllocation.enabled`를 true로 설정하면 기능이 활성화되며, `spark.dynamicAllocation.shuffleTracking.enabled`를 통해 셔플 파일 추적 기반 동적 할당을 사용할 수 있다.

Executor가 유휴 상태일 때 얼마 후에 반환할지를 결정하는 옵션이 `spark.dynamicAllocation.executorIdleTimeout`이며, 반대로 새 Executor를 요청하는 시점을 제어하는 옵션이 `spark.dynamicAllocation.schedulerBacklogTimeout`이다.

또한 최소·최대·초기 Executor 수를 각각 `spark.dynamicAllocation.minExecutors`, `spark.dynamicAllocation.maxExecutors`, `spark.dynamicAllocation.initialExecutors`로 지정할 수 있다. `spark.dynamicAllocation.executorAllocationRatio`는 Executor 증가 속도를 조절하는 역할을 한다.

### Spark Scheduler
---
Spark Scheduler는 **하나의 Spark Application 내부에서 여러 Job 간에 리소스를 분배하는 정책**이다. Spark Application들 간의 리소스 분배는 YARN과 같은 리소스 매니저가 담당하지만, Application 내부의 스케줄링은 Spark Scheduler의 역할이다.

Spark Scheduler에는 두 가지 모드가 존재한다. 기본값은 FIFO 방식으로, 먼저 제출된 Job이 리소스를 우선적으로 할당받는다. 이 방식은 단순하지만, 뒤에 들어온 Job이 오래 대기해야 할 수 있다.

**FAIR Scheduler**
FAIR Scheduler는 라운드 로빈 방식으로 Job 간에 리소스를 고르게 분배한다. 이를 통해 여러 Job이 동시에 진행되며, 특정 Job이 전체 리소스를 독점하는 상황을 방지할 수 있다.

FAIR Scheduler에서는 Pool이라는 개념을 사용해 Job들을 그룹화할 수 있다. Pool 단위로 리소스를 분배하며, 각 Pool 내부에서도 FIFO 또는 FAIR 정책을 적용할 수 있다. 이를 통해 우선순위를 고려한 보다 정교한 리소스 관리가 가능하다.

**Scheduler를 활용한 병렬성 증대**
Spark에서 병렬성을 높이기 위해서는 Thread 활용과 함께 적절한 스케줄링 전략이 필요하다. 특히 FAIR Scheduler를 사용하는 경우, 여러 Job을 동시에 실행하면서도 리소스를 균형 있게 사용할 수 있어 병렬성 증대 효과가 크다.

이를 위해 `spark.scheduler.mode`를 FIFO 대신 FAIR로 설정할 수 있으며, FAIR 모드에서는 `spark.scheduler.allocation.file`을 통해 Pool 설정 파일을 정의해야 한다.





## Spark Shuffling 최적화
---




## Spark Partition 학습
---




## Spark ML 소개와 ML 모델 빌딩
---




## ML Pipeline과 Tuning 소개와 실습
---



