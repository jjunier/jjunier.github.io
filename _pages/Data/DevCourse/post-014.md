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
- **Broadcast Variable**로 **셔플(Shuffle)을 최소화**하고(Closure 대비 차이 이해 포함) 룩업/조인 **성능을 개선**한다.
- **리소스·스케줄링·메모리 관리(Driver/Executor, Unified/Off-Heap, OOM)**를 이해하고 **핵심 튜닝 포인트**를 정리한다.
- **캐싱, Pushdown, 파티션 재조정(repartition/coalesce), 힌트, AQE(스큐 조인 포함)**를 통해 **실행 계획 최적화**를 적용한다.


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

리소스 설정은 spark-submit 명령어를 통해 `--num-executors`, `--executor-cores`, 
`--executor-memory`와 같은 옵션으로 제어할 수 있다.

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
Dynamic Allocation은 여러 환경 변수를 통해 세밀하게 제어할 수 있다. 
`spark.dynamicAllocation.enabled`를 true로 설정하면 기능이 활성화되며, 
`spark.dynamicAllocation.shuffleTracking.enabled`를 통해 
셔플 파일 추적 기반 동적 할당을 사용할 수 있다.

Executor가 유휴 상태일 때 얼마 후에 반환할지를 결정하는 옵션이 
`spark.dynamicAllocation.executorIdleTimeout`이며, 
반대로 새 Executor를 요청하는 시점을 제어하는 옵션이 
`spark.dynamicAllocation.schedulerBacklogTimeout`이다.

또한 최소·최대·초기 Executor 수를 각각 
`spark.dynamicAllocation.minExecutors`, `spark.dynamicAllocation.maxExecutors`, 
`spark.dynamicAllocation.initialExecutors`로 지정할 수 있다. 
`spark.dynamicAllocation.executorAllocationRatio`는 
Executor 증가 속도를 조절하는 역할을 한다.

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

### Spark Application에서 Driver의 역할
---
Spark 애플리케이션은 기본적으로 **1개의 `Driver`와 1개 이상의 `Executor`**로 구성된다. 이 중 Driver는 애플리케이션의 시작부터 종료까지 전체 실행을 총괄하는 핵심 컴포넌트이다.

Driver는 `main`함수를 실행하여 `SparkSession`과 `SparkContext`를 생성하고, 사용자가 작성한 코드를 분석하여 **Task 단위로 분해한 뒤 DAG(Directed Acyclic Graph) 를 생성**한다. 이후 DAG는 Logical Plan, Physical Plan, Execution Plan으로 변환되며, 이 과정에서 리소스 매니저와 협력하여 Executor에 Task를 분배하고 실행 상태를 관리한다.

또한 Job, Stage, Task의 실행 정보는 **Spark Web UI(기본 4040 포트)를 통해 확인**할 수 있다.

다만 Task 수가 과도하게 많아질 경우, 메타데이터를 관리하는 Driver의 메모리 사용량이 증가하면서 **Driver OOM이 발생할 수 있다는 점은 주의**해야 한다.

**Executor 메모리 관리 방식의 변화**
Spark 초기 버전에서는 Execution과 Storage 메모리를 **고정된 비율로 나누는 Static Memory Management 방식을 사용**했다. 이 방식은 구조는 단순하지만, 한쪽 메모리가 남아도 다른 쪽에서 사용할 수 없어 메모리 활용 효율이 낮았다.

이를 개선하기 위해 Spark 1.6 이후부터는 **Unified Memory Manager가 도입**되었다.

**Unified Memory Management의 동작 원리**
Unified Memory Manager는 Execution 메모리와 Storage 메모리를 유동적으로 공유한다. 기본적으로 실행 중인 Task를 기준으로 메모리를 공정하게 할당하며, 필요 시 한 영역의 남는 메모리를 다른 영역에서 사용할 수 있다.

Execution 메모리가 부족해지면 Storage Memory Pool의 여유 공간을 사용하고, 반대로 DataFrame이나 RDD 캐싱을 위한 Storage 메모리가 부족할 경우 Execution 메모리의 일부를 활용한다. 이때 `spark.memory.storageFraction`은 초기 경계 비율로 사용되며, 메모리가 모두 차기 시작하면 해당 경계를 기준으로 eviction이 발생한다.

**메모리 부족 상황과 Spill**
Executor 내에서 더 이상 사용할 수 있는 메모리가 없을 경우, Spark는 데이터를 메모리에서 디스크로 spill하여 처리한다. 이는 Job 실패를 방지할 수 있지만, 디스크 I/O로 인해 성능 저하를 유발한다.

만약 디스크 spill조차 불가능한 상황이 되면, 결국 **OOM(Out Of Memory) 이 발생하며 Executor 또는 Job이 실패**하게 된다.

### Off-Heap Memory
---
Spark는 기본적으로 JVM Heap(On-Heap) 메모리에서 가장 잘 동작하도록 설계되어 있다. 하지만 **JVM Heap은 Garbage Collection(GC)의 대상**이 되며, Heap 크기가 커질수록 GC로 인한 성능 비용 역시 증가한다는 한계를 가진다.

이러한 문제를 완화하기 위해 Spark는 JVM 외부 메모리, 즉 Off-Heap 메모리를 함께 사용할 수 있도록 지원한다. Off-Heap 메모리는 GC의 영향을 받지 않기 때문에, 메모리 사용량이 큰 워크로드에서 성능 안정성을 높이는 데 유리하다.

Spark에서 Off-Heap 메모리를 사용하려면 다음 설정이 필요하다.

- `spark.memory.offHeap.enabled = true`
- `spark.memory.offHeap.size`에 사용할 메모리 크기 지정

이 외에도 Executor 프로세스가 사용하는 **Overhead 메모리 역시 JVM Heap 외부에서 관리**된다.

### Spark 3.x의 Off-Heap Memory 구조
---
Spark 3.x부터는 Off-Heap 메모리 활용이 더욱 적극적으로 최적화되었다.
대표적인 예가 **Project Tungsten을 기반으로 한 메모리 관리 방식**이다.

Spark 3.x는 JVM에 의존하지 않고 직접 메모리를 관리할 수 있으며, 이 Off-Heap 메모리를 주로 **DataFrame 연산에 사용**한다. 이를 통해 GC 발생 빈도를 줄이고, 대규모 데이터 처리 시 성능을 보다 안정적으로 유지할 수 있다.

Spark 3.x 기준으로 Executor가 사용하는 Off-Heap 관련 메모리는 다음과 같이 정리할 수 있다.

- `spark.executor.memoryOverhead`
- `spark.memory.offHeap.size`

즉, Executor Heap 메모리를 늘리지 않고도 `spark.memory.offHeap.size` 설정을 통해 Off-Heap 메모리만 독립적으로 확장할 수 있다.

### Spark에서 발생하는 메모리 이슈
---
Spark에서 발생하는 메모리 문제는 크게 두 가지로 나뉜다.

- **Driver OOM**
- **Executor OOM**

두 경우는 원인과 대응 방식이 전혀 다르기 때문에, 구분해서 이해하는 것이 중요하다.

**Driver OOM이 발생하는 대표적인 경우**
Driver는 모든 메타데이터와 실행 계획을 관리하기 때문에, 특정 패턴에서 메모리 사용량이 급증할 수 있다.

대표적인 Driver OOM 케이스는 다음과 같다.

- 대규모 데이터셋에 대해 `collect()` 호출
- 큰 데이터셋을 대상으로 한 **Broadcast Join**
- Python, R 등 JVM 외 언어로 작성된 코드
- 지나치게 많은 Task 생성

특히 `collect()`나 Broadcast Join은 데이터를 Driver로 직접 가져오는 구조이기 때문에, 데이터 크기에 대한 고려 없이 사용하면 매우 위험하다.

**Executor OOM이 발생하는 대표적인 경우**

Executor OOM은 주로 데이터 분포와 병렬성 설정 문제에서 발생한다.

대표적인 원인은 다음과 같다.

- `spark.executor.cores` 값이 지나치게 큰 경우
    → 하나의 Executor에서 동시에 너무 많은 Task 실행 (High Concurrency)

- Data Skew로 인해 특정 Partition이 과도하게 커진 경우
    → 일부 Task에서 메모리 집중 사용 발생

이러한 상황에서는 Executor 메모리 증설보다는,
**Partition 전략, Join 방식, Executor 자원 설정을 함께 점검하는 것이 효과적**이다.


### JVM과 Python 간의 통신
---
**Pyspark Driver의 구조**
PySpark 애플리케이션의 Driver는 단일 프로세스가 아닌 두 개의 프로세스로 구성된다.

- Python 프로세스
- JVM 프로세스

Spark 자체는 JVM 기반 애플리케이션이지만, PySpark는 Python 코드를 실행해야 하므로 **JVM과 Python 프로세스가 병렬적으로 동작**한다. 이 구조로 인해 PySpark는 순수 Spark(Scala/Java)와는 다른 메모리 특성을 가진다.

![Pyspark Driver](/assets/img/contents/pyspark_driver.png "Pyspark Driver")

**Pyspark 메모리 구조**
Spark는 JVM 위에서 동작하지만, **PySpark의 Python 코드는 JVM 내부에서 직접 실행되지 않는다.** 따라서 Python 코드는 JVM Heap 메모리를 직접 사용할 수 없으며, 별도의 메모리 영역을 사용하게 된다.

이를 위해 Spark는 PySpark 전용 메모리 설정을 제공한다.

- `spark.executor.pyspark.memory`
    → Python 프로세스가 사용하는 메모리

- `spark.python.worker.memory`
    → JVM과 Python 간 통신을 담당하는 Py4J가 사용하는 메모리

이 두 설정은 PySpark 성능과 안정성에 직접적인 영향을 미친다.

![Pyspark Memory](/assets/img/contents/pyspark_memory.png "Pyspark Memory")

PySpark는 기본적으로 Executor의 overhead memory를 사용한다.

`spark.executor.pyspark.memor`y가 설정되면, Python 프로세스가 사용할 수 있는 메모리 크기는 해당 값으로 고정된다. 다만 이 설정은 주로 외부 Python 라이브러리나 **사용자 정의 Python 함수를 사용하는 경우에 필요**하며, 기본적으로는 명시적으로 설정되지 않는다.

- 기본값: 512MB (512m)
- JVM과 Python 프로세스 간 통신을 담당하는 Py4J가 사용할 수 있는 최대 메모리
- 해당 크기를 초과하면 **Disk Spill 발생**

`spark.executor.pyspark.memory`
    → Python 프로세스 자체가 사용할 수 있는 메모리 크기

`spark.python.worker.memory`
    → JVM 내부에서 관리되는 Python 오브젝트의 최대 메모리 크기

**Spark와 Python 간의 통신 방식**
Spark와 Python은 Py4J라는 프레임워크를 통해 데이터를 주고받는다.
Py4J는 Python과 JVM 간의 데이터 교환을 담당하며, PySpark의 핵심 구성 요소 중 하나이다.

DataFrame이나 RDD 연산 중 Python 코드가 사용되면, 해당 로직은 별도의 Python 프로세스에서 실행된다. 이 과정에서 **Partition 단위의 데이터가 Python 프로세스로 전달**되며, 이 데이터 이동 비용이 PySpark 성능에 영향을 줄 수 있다.

**Spark와 UDF(User Defined Function)**
Spark에서 UDF는 작성 언어와 방식에 따라 성능 특성이 크게 달라진다:

- Java / Scala UDF
    → JVM 내부에서 실행되어 성능상 가장 유리
- Python UDF
    → Py4J를 통한 데이터 직렬화/역직렬화 비용 발생
- Pandas UDF (Vectorized UDF)
    → PyArrow 기반, 컬럼 단위 처리로 성능 개선

특히 Pandas UDF는 Vectorized 방식으로 동작하며, PyArrow를 활용해 JVM ↔ Python 간 데이터 전송 비용을 줄인다. PySpark 환경에서 Python UDF를 사용해야 한다면, 가능하다면 **Pandas UDF를 우선 고려하는 것이 바람직**하다.

### Caching
---
Caching은 **자주 사용되는 DataFrame을 메모리에 유지하여 반복 연산 시 처리 속도를 향상시키는 기법**이다. 동일한 DataFrame을 여러 번 사용하는 경우, 매번 계산을 다시 수행하는 대신 캐시된 데이터를 재사용함으로써 성능을 크게 개선할 수 있다.

다만, 캐싱했다고 해서 항상 성능이 좋아지는 것은 아니다. 실제로 해당 DataFrame이 메모리에 유지되고 있는지 확인해야 하며, 경우에 따라서는 다시 계산하는 것이 더 빠른 상황도 존재한다. 또한 캐싱은 메모리 사용량을 증가시키므로, 모든 DataFrame을 무분별하게 캐싱하는 것은 바람직하지 않다.

**DataFrame을 캐싱하는 방법 (1)**
Spark에서 DataFrame을 캐싱하는 방법은 크게 두 가지가 있다.

- cache()
- persist()

두 방법 모두 DataFrame을 메모리, 디스크, 또는 Off-Heap 영역에 보관할 수 있으며, Lazy Execution 방식으로 동작한다. 즉, 실제 액션이 실행되기 전까지는 캐싱이 수행되지 않는다.

또한 캐싱은 항상 Partition 단위로 이루어지며, 하나의 파티션이 부분적으로만 캐싱되는 일은 없다.

**DataFrame을 캐싱하는 방법 (2)**
persist()는 인자를 통해 캐싱 방식을 보다 세밀하게 제어할 수 있다.

- `useMemory` : 메모리 사용 여부
- `useDisk` : 디스크 사용 여부
- `useOffHeap` : Off-Heap 사용 여부 (사전 설정 필요)
- `deserialized`
    - True : CPU 연산 감소, 메모리 사용 증가
    - False : 메모리 절약, CPU 연산 증가
    - 메모리 캐싱에서만 사용 가능
- `replication`
    - 서로 다른 Executor에 저장할 복제본 개수

이 설정을 통해 메모리와 CPU 자원 간의 트레이드오프를 조절할 수 있다.

**DataFrame을 캐싱하는 방법 (3)**
persist()에서 자주 사용되는 설정 조합은 상수 형태로 제공된다.

- `DISK_ONLY`
- `MEMORY_ONLY`
- `MEMORY_AND_DISK`
- `MEMORY_ONLY_SER`
- `MEMORY_AND_DISK_SER`
- `OFF_HEAP`
- `MEMORY_ONLY_2`
- `MEMORY_ONLY_3`

상수를 사용하면 캐싱 전략을 간단하면서도 명확하게 표현할 수 있다.

**DataFrame을 캐싱하는 방법 (4)**
기본적으로 persist()는 캐싱된 DataFrame을 메모리와 디스크에 저장하며, 필요 시 복제도 수행한다.

`cache()`는 persist()의 단순화된 버전으로, 내부적으로 다음과 같은 설정을 사용한다.

- useDisk = false
- useMemory = true
- useOffHeap = false
- deserialized = true
- replication = 1

즉, cache()는 메모리 기반 캐싱을 간단히 적용하고 싶을 때 적합하다.

**Spark SQL을 이용한 Caching**
DataFrame API 외에도 Spark SQL을 통해 테이블 단위 캐싱이 가능하다.

- `CACHE TABLE table_name`
- `CACHE LAZY TABLE table_name`
- `UNCACHE TABLE table_name`

```python
spark.sql("cache table table_name")
spark.sql("cache lazy table table_name")
spark.sql("uncache table table_name")
```

이를 통해 SQL 기반 워크플로우에서도 캐싱 전략을 일관되게 적용할 수 있다.

**Caching을 해제하는 방법**
캐싱된 데이터는 필요 없어졌을 때 반드시 해제하는 것이 중요하다.

- DataFrame.unpersist()
    → LRU(Least Recently Used) 정책 기반
- UNCACHE TABLE table_name
- spark.catalog.isCached("table_name")
- spark.catalog.clearCache()

```python
DataFrame.unpersist (LRU - Least Recently Used)
spark.sql("uncache table table_name")
spark.catalog.isCached("table_name")
spark.catalog.clearCache()
```

적절한 캐시 해제는 전체 애플리케이션의 메모리 안정성을 높인다.

**Caching 관련 Best Practices**
캐싱을 효과적으로 사용하기 위해서는 몇 가지 원칙을 지키는 것이 좋다.

- 캐싱된 DataFrame이 명확하게 재사용되도록 변수로 분리
- 컬럼 수가 많다면 필요한 컬럼만 선택해서 캐싱
- 더 이상 사용하지 않는 경우 즉시 uncache
- Parquet 등 컬럼 기반 포맷의 대규모 데이터셋은
    → 매번 다시 읽는 것이 캐싱보다 빠를 수 있음
- 캐싱 대상은 소수의 핵심 DataFrame으로 제한
- 대형 DataFrame 캐싱은 지양
- 캐싱을 만능 해결책으로 신뢰하지 말 것

### Filter (Predicate) Pushdown
---
Spark에서 대용량 데이터를 처리할 때 성능을 좌우하는 가장 중요한 요소 중 하나는 얼마나 적은 데이터를 읽느냐이다. 이를 위해 Spark는 여러 최적화 기법을 제공하는데, 그중 대표적인 것이 **Filter(Predicate) Pushdown과 Partition Pruning**를 사용한다.

Filter Pushdown은 데이터를 모두 읽은 뒤 필터링하는 방식이 아니라, 데이터 소스에서 읽는 시점에 필터 조건을 적용하여 불필요한 데이터를 아예 로드하지 않는 최적화 기법이다.

이 방식은 모든 데이터 소스에서 지원되지는 않으며, 대표적으로 Parquet 포맷에서 컬럼 통계 정보(min/max 등)가 존재하는 경우에만 효과적으로 동작한다. Filter Pushdown이 적용되면 I/O 자체가 줄어들기 때문에 성능 개선 효과가 크다.

### Partition Pruning이란?
---
Partition Pruning은 Spark Optimizer가 필요한 파티션만 선택적으로 읽도록 하는 최적화 기법이다. Optimizer는 쿼리를 분석해 실제로 필요한 데이터가 들어 있는 파티션과 그렇지 않은 파티션을 구분하고, 불필요한 파티션은 스캔 대상에서 제외한다.

이 최적화는 Spark의 Logical Plan Optimization 단계에서 수행된다.

![Partition Pruning](/assets/img/contents/partition_pruning.png "Partition Pruning")

### Static Partition Pruning
---
Static Partition Pruning은 쿼리 실행 전에 어떤 파티션을 읽을지 명확하게 알 수 있는 경우에 적용된다. 주로 테이블이 특정 컬럼을 기준으로 파티셔닝되어 있고, 쿼리의 필터 조건에 해당 파티션 컬럼이 직접 사용되는 경우이다.

하지만 현실적인 데이터 모델에서는 파티셔닝이 보통 Fact 테이블에 적용되어 있고, 필터 조건은 Dimension 테이블에 걸리는 경우가 많아 Static 방식만으로는 한계가 있다.

### Dynamic Partition Pruning
---
Dynamic Partition Pruning은 이러한 한계를 보완하기 위한 기법이다. 파티션되지 않은 테이블(Dimension)에 적용된 필터 조건을 실행 시점에 **파티션 테이블(Fact)에 동적으로 전달**하여 필요한 파티션만 읽도록 한다.

특히 Dimension 테이블이 작아 Broadcast Join까지 활용된다면 성능 개선 효과는 더욱 커진다.

Dynamic Partition Pruning은 기본적으로 활성화되어 있으며, 다음 설정으로 확인할 수 있다.


## Spark Shuffling 최적화
---
### Repartition을 사용하는 이유
---
`repartition`은 말 그대로 **데이터프레임의 파티션을 다시 나누는 작업**이다. 이 연산이 필요한 대표적인 이유는 다음과 같다.

첫째, **병렬성을 높이기 위해서다.** 파티션 수가 너무 적으면 각 태스크가 처리해야 할 데이터 양이 커지고, 클러스터의 리소스를 충분히 활용하지 못한다. 이럴 때 파티션 수를 늘려주면 병렬 처리가 가능해진다.

둘째, **지나치게 큰 파티션이나 Skew(쏠림) 파티션을 조정하기 위해서다.** 특정 키에 데이터가 몰려 있는 경우 일부 태스크만 오래 걸리게 되는데, repartition을 통해 이를 완화할 수 있다.

셋째, **분석 패턴에 맞게 데이터를 재분배하기 위해서다.** 예를 들어 어떤 DataFrame을 특정 컬럼 기준으로 자주 그룹핑하거나 필터링한다면, 그 컬럼 기준으로 미리 파티션을 나눠 저장해두는 것이 효율적이다. 이를 “Write once, read many” 패턴이라고 부르며, 이와 유사한 개념이 바로 **Bucketing**이다.

### Repartition 방식의 특징과 주의점
---
Spark에서는 repartition 방식으로 크게 두 가지를 제공한다.

- `repartition`
- `repartitionByRange`

이 둘의 공통점은 항상 Shuffling이 발생한다는 것이다. 즉, 네트워크 I/O와 디스크 사용이 뒤따르며, 비용이 상당하다. 따라서 repartition은 “그럴듯해 보이니까” 쓰는 연산이 아니라, 명확한 이유가 있을 때만 사용해야 한다.

실무에서는 종종 repartition이 아무 근거 없이 사용되어 오히려 전체 처리 시간이 늘어나고 비용만 증가하는 경우를 본다. 불필요한 `count()`, `distinct()`, 중복 제거 연산이 성능을 악화시키는 것과 비슷한 맥락이다.

또 하나 주의할 점은, **컬럼을 기준으로 repartition한다고 해서 항상 균등한 파티션 크기가 보장되지는 않는다**는 것이다. 데이터 분포 자체가 불균형하다면, 결과 파티션 역시 Skew를 가질 수 있다.

마지막으로, repartition은 파티션 수를 줄이는 용도로는 적합하지 않다. 줄이고 싶다면 반드시 `coalesce`를 사용해야 한다.

repartition(numPartitions, *cols)

`repartition`은 Hash 기반 파티셔닝을 사용한다. 사용 예시는 다음과 같다.

- `repartition(5)`
- `repartition(5, "city")`
- `repartition(5, "city", "zipcode")`
- `repartition("city")`
- `repartition("city", "zipcode")`

컬럼을 지정하지 않으면 전체 데이터를 랜덤하게 섞어 파티션을 나누고, 컬럼을 지정하면 해당 컬럼의 해시 값을 기준으로 파티션이 결정된다. 다만 앞서 언급했듯, 해시 기반이라고 해서 파티션 크기가 항상 균등해지는 것은 아니다.

### repartitionByRange(numPartitions, *cols)
---
`repartitionByRange`는 **지정한 컬럼 값의 범위(range)**를 기준으로 파티션을 나눈다. 내부적으로는 데이터 샘플링을 통해 경계를 정하기 때문에, **실행할 때마다 결과가 달라질 수 있는 비결정적(Nondeterministic) 연산**이다.

사용법은 repartition과 거의 동일하지만, 값의 범위가 의미 있는 컬럼(예: 날짜, 숫자 ID 등)에 특히 적합하다. 다만 이 역시 Shuffling이 발생하므로, 사용 전 반드시 비용 대비 효과를 고민해야 한다.

### Coalesce가 필요한 경우
---
`coalesce`는 repartition과 목적이 다르다. 이 연산은 파티션 수를 줄이는 데에만 사용한다.

가장 큰 특징은 Shuffling을 발생시키지 않는다는 점이다. 기존의 로컬 파티션들을 그대로 병합하기 때문에 비용은 적지만, 그만큼 Skew 파티션이 생길 가능성도 높다.

또한 `coalesce` 역시 컬럼 기반으로 사용할 수는 있지만, 이 경우에도 균등한 파티션 크기는 보장되지 않는다. 따라서 대규모 후처리 단계에서 “결과 파일 수를 줄이고 싶을 때”처럼 명확한 목적이 있을 때 사용하는 것이 바람직하다.

### DataFrame 힌트에 대한 간단한 정리
---
마지막으로, Spark에서는 **DataFrame 힌트(hint)**를 통해 Spark SQL Optimizer에게 실행 계획에 대한 “제안”을 할 수 있다. 이는 기본 최적화 전략을 완전히 바꾸기보다는, 특정 상황에서 더 적합한 실행 계획을 유도하기 위한 장치다.

힌트는 크게 두 가지로 나뉜다.

- Partitioning 관련 힌트
- Join 관련 힌트

복잡한 조인이나 대규모 데이터 처리에서, 힌트를 적절히 활용하면 Spark가 더 효율적인 Execution Plan을 선택하도록 도울 수 있다.

### DataFrame Partitioning 관련 힌트들
---
Spark는 파티션 전략과 관련된 여러 힌트를 제공한다. 이 힌트들은 Execution Plan을 생성하는 과정에서 파티션을 어떻게 다룰지에 대한 방향성을 Optimizer에게 전달한다.

대표적인 파티셔닝 관련 힌트는 다음과 같다.

- `COALESCE`
- `REPARTITION`
- `REPARTITION_BY_RANGE`
- `REBALANCE`

이 힌트들은 특히 **DataFrame을 테이블이나 파일 형태로 저장할 때 매우 유용**하다. 저장 단계에서 파티션 전략을 잘 잡아두면, 이후 반복적인 조회(Read)가 훨씬 효율적으로 이루어진다.

그중 REBALANCE는 파일 크기를 최대한 비슷하게 맞춰 저장하는 데 목적이 있다. 다만 이 기능은 **AQE(Adaptive Query Execution)**가 활성화되어 있어야 효과적으로 동작한다. AQE를 통해 실행 시점에 파티션을 재조정하면서, 결과 파일들이 특정 파티션에 몰리지 않도록 균형을 맞춘다.

예를 들어, 조인 이후 결과 파티션 수를 줄이고 싶다면 다음과 같이 힌트를 줄 수 있다.

```scala
df1.join(df2, "id", "inner")
  .hint("COALESCE", 3)
```
이 코드는 조인 결과를 3개의 파티션으로 병합하라는 힌트를 Optimizer에게 전달한다. 중요한 점은, 이는 **강제(force)가 아니라 제안(hint)**이라는 점이다. Spark는 전체 실행 계획과 비용을 고려해 힌트를 무시할 수도 있다.

### DataFrame Join 관련 힌트들
---
힌트가 가장 많이 사용되는 영역은 단연 **Join**이다. Spark는 데이터 크기와 통계 정보를 기반으로 자동으로 조인 전략을 선택하지만, 사용자가 데이터 특성을 더 잘 알고 있는 경우 힌트를 통해 이를 유도할 수 있다.

**Broadcast 계열 힌트**

- `BROADCAST`
- `BROADCASTJOIN`
- `MAPJOIN`

이 힌트들은 Broadcast Join 사용을 제안한다. 한쪽 테이블이 충분히 작을 경우, 이를 모든 Executor에 복제해 네트워크 Shuffle을 피하는 전략이다. 소규모 Dimension 테이블과의 조인에서 매우 효과적이다.

**Merge 계열 힌트**

- `MERGE`
- `SHUFFLE_MERGE`
- `MERGEJOIN`

이 힌트들은 Shuffle Merge Join 사용을 제안한다. 이는 Spark의 기본 조인 전략이기도 하며, 양쪽 데이터가 모두 크고 조인 키 기준으로 정렬이 가능한 경우에 적합하다.

**Shuffle Hash Join 힌트**

- `SHUFFLE_HASH`

Shuffle Hash Join을 사용하도록 제안하는 힌트다. 다만 제약이 명확한데, Full Outer Join에서는 사용할 수 없다. 특정 상황에서는 Merge Join보다 빠를 수 있지만, 메모리 사용량이 늘어날 수 있어 주의가 필요하다.

**Shuffle Replicate NL 힌트**

- `SHUFFLE_REPLICATE_NL`

이 힌트는 **Shuffle-and-replicate 방식의 Nested Loop Join, 즉 사실상 Cross Join을 유도**한다. 조인 조건이 없거나 매우 특수한 경우에만 사용해야 하며, 데이터 크기가 크면 비용이 폭발적으로 증가한다.

한 가지 중요한 규칙은, 여러 개의 조인 힌트가 동시에 사용될 경우 우선순위가 존재한다는 점이다. 일반적으로 위에서 아래로 갈수록 우선순위가 낮아지며, Spark는 가장 우선순위가 높은 힌트를 먼저 고려한다.

예를 들어 Spark SQL에서는 다음과 같이 조인 힌트를 명시할 수 있다.

```sql
SELECT /*+ MERGE(df2) */ *
FROM df1
JOIN df2
  ON df1.order_month = df2.year_month
```

이 쿼리는 df2와의 조인에서 Merge Join을 우선적으로 고려하라는 의미다.

### DataFrame 힌트 사용법 정리
---
힌트는 Spark SQL과 DataFrame API 양쪽에서 모두 사용할 수 있다.

**Spark SQL에서의 힌트**

Spark SQL에서는 주석 형태로 힌트를 작성한다.

```sql
/*+ hint [, … ] */
```

대표적인 예시는 다음과 같다.

```sql
SELECT /*+ REPARTITION(3) */ *
FROM table
```

또는 조인 시 Broadcast Join을 유도하는 경우,

```sql
SELECT /*+ BROADCAST(table1) */ *
FROM table1
JOIN table2
  ON table1.key = table2.key

```

**DataFrame API에서의 힌트**

DataFrame API에서는 .hint() 메소드를 사용한다.

```scala
val joinDf =
  df1.join(df2, "id", "inner")
     .hint("COALESCE", 3)
```

또는 조인 대상 중 하나에만 Broadcast 힌트를 주고, 결과 파티션까지 함께 제어할 수도 있다.

```scala
val joinDf =
  df1.join(df2.hint("broadcast"), "id", "inner")
     .hint("COALESCE", 3)
```

### Spark Optimization의 역사
---
**Spark 1.x: Catalyst Optimizer와 Tungsten Project**
Spark 1.x 시절의 최적화는 크게 두 축으로 이루어져 있었다.

먼저 Catalyst Optimizer다. Catalyst는 규칙 기반(rule-based) 최적화 엔진로, SQL이나 DataFrame 연산을 논리적으로 변환하는 역할을 했다. 대표적인 최적화로는 조건절을 최대한 아래로 밀어내는 Predicate Pushdown, 필요한 컬럼만 읽도록 하는 Projection Pushdown 등이 있다. 이 단계에서는 “어떤 순서로 연산을 수행할 것인가”에 초점이 맞춰져 있었다.

두 번째는 Tungsten Project다. 이는 성능의 발목을 잡던 JVM 특유의 한계, 특히 GC 비용을 줄이기 위한 시도였다. Spark는 Tungsten을 통해 Off-Heap 메모리 관리를 직접 수행하고, 코드 생성(Code Generation)을 통해 CPU 친화적인 실행을 지향했다. 즉, “어떻게 빠르게 실행할 것인가”에 대한 해답이었다.

**Spark 2.x: CBO (Cost-Based Optimizer)**
Spark 2.x에 들어서면서 한 단계 더 진화한 개념이 등장한다. 바로 **CBO(Cost-Based Optimizer)**다.

CBO는 DataFrame 및 테이블의 통계 정보를 기반으로, 여러 실행 계획 중 가장 비용이 낮을 것으로 예상되는 plan을 선택한다. 여기에는 다음과 같은 정보들이 활용된다.

- 전체 데이터 크기
- 레코드 수
- 컬럼별 최소값 / 최대값
- 히스토그램 등 분포 정보

즉, Spark는 더 이상 “항상 같은 규칙”이 아니라, 데이터의 특성에 따라 다른 실행 계획을 선택할 수 있게 되었다. 다만 이 방식에는 한계가 있었는데, 통계 정보는 대부분 **실행 전(parsing time)**에 수집된다는 점이다.

### AQE 이전의 세계
---
다음과 같은 단순한 GROUP BY 쿼리를 생각해보자.

```sql
SELECT sku, SUM(price) AS sales
FROM order
GROUP BY sku;
```

AQE가 없던 시절, Spark는 이 쿼리를 실행하면서 고정된 실행 계획을 만든다.

이 쿼리는 보통 두 개의 Stage를 생성한다:

1. 데이터를 읽고 Shuffle을 수행하는 Stage
2. Shuffle 결과를 받아 최종 Aggregation을 수행하는 Stage

이때 Shuffle 이후 생성되는 파티션 수는 spark.sql.shuffle.partitions 설정 값에 의해 고정된다. 문제는, 이 시점에서는 실제 데이터 크기나 분포를 정확히 알기 어렵다는 점이다.

**spark.sql.shuffle.partitions의 한계**
spark.sql.shuffle.partitions는 Spark 성능 튜닝에서 가장 자주 언급되는 설정 중 하나다. MapReduce 시절의 mapreduce.job.reduces와 거의 동일한 역할을 한다.

하지만 이 값 하나로 모든 상황을 커버하기는 매우 어렵다.

- 파티션 수가 너무 적으면,
    → 병렬성이 낮아지고, OOM이나 디스크 스필 가능성이 커진다.

- 파티션 수가 너무 많으면,
    → Task 생성과 스케줄링 오버헤드가 증가하고, 잦은 네트워크 I/O로 병목이 발생한다.

결국 핵심 질문은 이것이다. “Spark가 알아서 상황에 맞게 파티션 수를 결정해줄 수는 없을까?”

**이 문제를 해결하기 위한 접근**
이 문제는 Spark만의 고민은 아니었다. 대용량 데이터베이스 분야에서는 이미 오래전부터 연구된 문제다.

Spark 3.0 이전, Intel Big Data 팀이 이 문제에 대한 프로토타입을 개발했고, 이후 Databricks와 협업하면서 Spark에 본격적으로 도입된다.

핵심 아이디어는 단순하다.
- Parsing time(실행 전) 최적화만으로는 충분하지 않다.
- Runtime(실행 중) 정보까지 활용해야 한다.

특히 UDF가 많이 사용되는 경우, 실행 전에는 비용을 예측하기가 거의 불가능하기 때문에 이 문제는 더 심각해진다.

### AQE란 무엇인가
---
AQE(Adaptive Query Execution)는 다음과 같이 정의된다.

> “실행 중(Runtime)에 수집한 통계 정보를 기반으로, 쿼리 실행 도중에 동적으로 최적화를 수행하는 방식”

즉, AQE는 모든 최적화 결정을 실제 실행 중에 관측한 정확한 통계 정보에 기반한다.

그렇다면 중요한 질문이 하나 남는다.
**언제 실행 중 통계를 수집하고, 언제 실행 계획을 바꾸는 것이 가장 좋을까?**

Spark의 실행 단계를 떠올려보면 답이 보인다.

```text
Query → Job → Stage → Task
```

**왜 Stage가 최적의 변경 시점인가**

Spark에서 Stage는 Shuffle이나 Broadcast를 기준으로 나뉜다. 그리고 Stage 경계에서는 다음과 같은 일이 발생한다.

- 중간 결과가 materialize 된다.
- 실제 파티션의 수와 크기를 정확히 알 수 있다.

즉, 이 시점은 데이터 분포를 추측이 아니라 사실로 알 수 있는 최초의 지점이다.

앞서 본 GROUP BY 쿼리를 다시 보면,

- Stage 0: Scan → Shuffle → 중간 SUM
- Stage 1: Shuffle 결과 → 최종 SUM

바로 이 두 번째 Stage가 시작되는 시점이, 최적화 방식을 바꾸기에 가장 이상적인 타이밍이다.

### AQE 이후의 세계
---
AQE가 활성화된 환경에서는, GROUP BY 쿼리의 두 번째 Stage 시작 시점에 **AQEShuffleRead**라는 새로운 메커니즘이 개입한다.

이를 통해 Spark는 다음과 같은 결정을 실행 중에 다시 내릴 수 있다.

**AQE가 특히 필요한 경우들**
AQE는 단순한 파티션 조정 기능이 아니다. 다음과 같은 고급 최적화를 가능하게 만든다.

- Shuffle 이후 파티션을 동적으로 병합(Coalescing)
    → Spark 3에서 도입

- 조인 전략을 실행 중에 전환
    → Spark 3.2에서 강화

- Skew Join을 동적으로 감지하고 최적화
    → Spark 3에서 도입

이제 Spark는 더 이상 “처음에 세운 계획을 끝까지 밀어붙이는 엔진”이 아니다.
실행하면서 보고, 판단하고, 전략을 바꾸는 엔진으로 진화했다.

## Spark Partition 학습
---
### Dynamically Optimizing Skew Joins란 무엇인가?
---
Spark 환경에서 대규모 데이터를 처리하다 보면 데이터 스큐(skew) 문제는 성능 저하의 가장 흔한 원인 중 하나이다.
Dynamically Optimizing Skew Joins는 이러한 스큐 문제를 **Spark AQE(Adaptive Query Execution)**가 런타임에 감지하고, 자동으로 최적화해주는 기능이다.

### 왜 Skew Join 최적화가 필요한가?
---
Skew Partition이 만드는 병목 현상

조인 연산 시 특정 파티션에 데이터가 과도하게 몰리면 다음과 같은 문제가 발생한다.

- 대부분의 태스크는 빠르게 끝나지만, 소수의 태스크만 유독 오래 실행한다
- 이 몇 개의 태스크 때문에 전체 Job / Stage 종료가 지연된다.
- 스큐 파티션이 메모리를 초과하면 Disk Spill 발생한다.
    - 디스크 I/O로 인해 성능이 급격히 저하된다.

즉, 클러스터 자원은 충분한데도 불구하고 불균형한 데이터 분포 하나 때문에 전체 성능이 무너지는 상황이 발생한다.

### AQE가 제시하는 해법
---
Spark의 AQE는 이러한 문제를 사전에 추측하지 않고, 실행 중에 관찰한 통계 정보를 기반으로 해결한다.

AQE 기반 Skew Join 최적화는 다음과 같은 전략을 따릅니다.

1. Skew Partition 존재 여부를 런타임에 탐지한다.
2. Skew Partition을 더 작은 여러 파티션으로 분할된다.
3. 조인 대상 반대편 파티션을 중복 생성한다.
4. 분할된 파티션 단위로 조인을 병렬 수행한다.

이 방식은 태스크 실행 시간을 고르게 분산시켜, 특정 태스크가 병목이 되는 상황을 제거한다.

### Dynamically Optimizing Skew Joins 동작 방식
---
1. Leaf Stage 실행
먼저 각 테이블의 Leaf Stage가 실행된다. 이 단계에서 Spark는 각 파티션의 실제 데이터 크기를 수집한다.

2. Skew Partition 감지 및 분할
AQE는 파티션 크기를 비교하여 비정상적으로 큰 파티션(skew partition) 을 감지한다.

감지된 skew partition은 Skew Reader를 통해 여러 개의 작은 파티션으로 재구성된다.

2. Skew Partition 감지 및 분할
AQE는 파티션 크기를 비교해 비정상적으로 큰 파티션(skew partition) 을 감지한다.

감지된 skew partition은 Skew Reader를 통해 여러 개의 작은 파티션으로 재구성됩니다.

4. 병렬 조인 수행

결과적으로 하나의 거대한 태스크가 아닌, 여러 개의 균등한 태스크로 구성된다.

이에 전체 Stage 실행 시간이 단축되고, Disk Spill 가능성도 크게 감소한다.

**주요 설정 파라미터**
Skew Join 최적화는 기본 설정으로도 동작하지만, 워크로드 특성에 따라 조정이 필요할 수 있다.

`spark.sql.adaptive.skewJoin.skewedPartitionFactor`
스큐 여부를 판단하는 비율 기준으로, 평균 파티션 크기 대비 몇 배 이상 크면 skew로 판단할지 결정한다. 기본값은 5이다.

`spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes`
절대 크기의 기준으로, 해당 바이트 크기를 초과해야 skew partition으로 간주한다. 기본값은 256MB이다.

이 두 조건을 모두 만족해야 **skew partition으로 인식**된다.