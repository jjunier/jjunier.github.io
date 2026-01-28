---
title: "[6기] 데브코스 DE WIL 11 | 빅데이터 처리 시스템 Hadoop & Spark"
tags:
    - DevCourse
    - Data Engineering
    - Spark
date: "2025-06-16"
thumbnail: "/assets/img/thumbnail/devcourse.png"
bookmark: true
---

## 이번 주 학습 목표 
--- 
- Hadoop과 Spark의 등장 배경 및 아키텍처(`HDFS`, `YARN`, `MapReduce`, `Spark Core`)를 이해하고, **대용량 데이터를 분산 환경에서 처리해야 하는 이유를 설명**할 수 있다.
- Spark의 실행 모델과 데이터 처리 방식(`DataFrame`, `Partition`, `Shuffle`, `Job·Stage·Task`)을 이해하고, **성능에 영향을 주는 핵심 요소를 파악**할 수 있다.
- `Parquet`, `Partitioning`, `Bucketing`과 같은 **저장 및 처리 최적화 기법을 이해**하고, 실무 관점에서 **효율적인 대규모 데이터 처리 파이프라인을 설계**할 수 있다.

## 빅데이터 정의와 예
---
빅데이터는 흔히 **단일 서버로 처리할 수 없는 규모의 데이터**로 정의된다. 이 정의는 2012년 아마존 클라우드 컨퍼런스에서 아마존의 데이터 사이언티스트 존 라우저(John Rauser)가 제시한 개념으로, 핵심은 데이터의 크기 자체보다 **분산 환경이 필요한가**에 있다.

예시로, `Pandas`와 같은 단일 머신 기반 도구로 데이터를 처리하려고 할 때, 메모리나 성능의 한계로 작업이 불가능하다면, 이는 빅데이터 문제로 볼 수 있다. 이 시점부터는 **여러 대의 서버를 활용하는 분산 처리 접근**이 필요해진다.

또 다른 관점에서의 빅데이터는 **기존의 소프트웨어로 처리할 수 없는 데이터**를 의미한다. 여기서 기존 소프트웨어란 Oracle이나 MySQL과 같은 전통적인 관계형 데이터베이스를 말한다.

이러한 시스템은 기본적으로 분산 환경을 두고 설계되지 않았으며, 성능 향상을 위해 `Scale-Up` 방식을 사용하였다. 즉, 메모리, CPU, 디스크를 추가하는 방식으로 한계를 극복하려 한다. 반면 빅데이터 환경에서는 **여러 노드를 추가**하는 `Scale-Out` 방식이 필요하다.

### 빅데이터의 4V
---
빅데이터는 흔히 4V로 설명되곤 한다:
- `Volume`: 데이터의 크기가 매우 큰가?
- `Velocity`: 데이터가 빠른 속도로 생성·처리되어야 하는가?
- `Variety`: 정형·반정형·비정형 데이터를 모두 포함하는가?
- `Veracity`: 데이터의 품질과 신뢰성이 확보되어 있는가?

이 네 가지 특성은 빅데이터 처리 시스템이 단순 저장을 넘어, 대규모 분산 처리와 품질 관리까지 고려해야 함을 의미한다.

### 빅데이터의 대표적 사례
---
**디바이스 데이터**
빅데이터는 다양한 디바이스에서 지속적으로 생성되는데, 대표적으로 모바일 디바이스에서 위치 정보와 같은 실시간성 데이터가 수집된다. 스마트 TV나 각종 IoT 센서에서는 사용 로그와 환경 데이터가 끊임없이 발생한다.

**웹 데이터**
웹은 가장 대표적인 빅데이터 환경 중 하나로, 전 세계에서는 수십 조 개 이상의 웹 페이지가 존재하며, 이는 사실상 방대한 지식의 집합체로 볼 수 있다. 

웹 검색 엔진 개발은 대규모 데이터 처리의 전형적 사례로, 웹 페이지를 크롤링하여 수집한 후, 중요도를 계산(PageRank)하고, 이를 인덱싱하여 사용자 요청에 빠르게 응답한다. 이러한 과정 속에서 구글은 빅데이터 기술 발전에 결정적인 역할을 기여해왔다.

또한 사용자 검색어와 클릭 로그 자체로도 매우 큰 데이터셋을 형성한다. 해당 데이터를 분석함으로써 개인화 서비스, 검색 트랜드 분석, 통계 기반 번역과 같은 다양한 부가 서비스를 개발할 수 있다. 최근에는 웹 데이터를 기반으로 한 **자연어 처리(NLP)** 대규모 모델의 학습 데이터로 활용되며, 그 중요성이 더욱 커지고 있다.


## 빅데이터 처리가 갖는 특징
---
빅데이터를 다루기 위해선 기존 데이터 처리 방식과는 다른 접근이 필요하다. 최우선적으로, **대용량 데이터를 손실 없이 저장할 수 있는 스토리지**가 필수적이다. 데이터 규모가 단일 서버의 디스크 용량을 넘어서는 경우가 많기 때문이다.

또한 빅데이터 처리는 **처리 시간이 오래 걸리는 경우가 많아 병렬 처리**가 요구된다. 하나의 프로세스로 데이터를 순차 처리하는 방식으로는 현실적인 시간 안에 결과를 얻기 어렵다.

더불어 빅데이터는 **비구조화 또는 반구조화 데이터**인 경우가 많다. 예를 들어 웹 로그 파일과 같은 데이터는 전통적인 SQL 기반 처리만으로는 한계가 있으며, 보다 유연한 처리 방식이 필요하다.

### 해결 방안
---
이러한 문제를 해결하기 위해서는 몇 가지 핵심 요소가 필요하다.
먼저, 대규모 데이터를 안정적으로 저장하기 위해 **분산 파일 시스템이 요구**된다. 여러 서버에 데이터를 분산 저장함으로써 용량 한계를 극복할 수 있다.

두 번째로, 대량의 데이터를 효율적으로 처리하기 위해 **병렬 처리가 가능한 분산 컴퓨팅 시스템**이 필요하다. 이를 통해 작업을 여러 노드에 분산시켜 처리 시간을 단축할 수 있다.

마지막으로, 비구조화 데이터를 다룰 수 있는 **유연한 데이터 처리 모델**이 필요하다. 이러한 요구사항을 종합하면, 결국 **다수의 컴퓨터로 구성된 분산 처리 프레임워크**가 필수적임을 알 수 있다.

### 대용량 분산 시스템의 특징
---
대용량 분산 시스템은 기본적으로 **분산 환경을 전제로 설계**되며, 하나 이상의 서버로 구성된다. 이 환경에서는 분산 파일 시스템과 분산 컴퓨팅 시스템이 함께 동작한다.

또한 일부 서버에 장애가 발생하더라도 전체 시스템이 정상적으로 동작해야 하는 `Fault Tolerance`가 중요하다. 마지막으로, 데이터 증가에 따라 서버를 추가하는 방식으로 손쉽게 확장할 수 있는 **Scale-out** 구조를 갖추는 것이 필수적이다.


## 하둡의 등장과 소개
---
Hadoop은 Doug Cutting이 구글 연구소에서 발표한 논문들을 기반으로 개발한 **오픈소스 분산 처리 프로젝트**이다. 그 출발점은 대규모 웹 데이터를 처리하기 위해 구글이 제안한 두 가지 핵심 기술이었다.

2003년에 발표된 The Google File System과 2004년에 공개된 MapReduce: Simplified Data Processing on Large Clusters 논문은 이후 분산 시스템 설계의 표준이 되었다.

초기 Hadoop은 오픈소스 검색 엔진 프로젝트인 Nutch의 하부 컴포넌트로 시작되었으며, 이름은 Doug Cutting의 아들이 가지고 있던 코끼리 인형에서 유래했다. 이후 2006년 Apache 재단의 톱레벨 프로젝트로 분리되며 본격적인 생태계를 형성하게 된다.

### Hadoop이란?
---
Hortonworks는 Hadoop을 **범용 하드웨어로 구성된 클러스터에서 대규모 데이터를 분산 저장하고 처리하기 위한 오픈소스 플랫폼**으로 정의한다. Hadoop 클러스터는 여러 대의 노드로 구성되며, 사용자 관점에서는 마치 하나의 거대한 컴퓨터처럼 동작한다.

실제로는 다수의 독립된 서버들이 복잡한 분산 소프트웨어에 의해 통제되며, 저장과 연산을 나누어 수행한다. 이를 통해 대용량 데이터를 효율적으로 처리할 수 있다.

### Hadoop의 발전 과정
---
Hadoop 1.0은 **HDFS 위에서 MapReduce가 직접 동작하는 구조**를 가지고 있었다. 이 환경 위에서 다양한 분산 컴퓨팅 언어와 프레임워크가 MapReduce 기반으로 개발되었다.

이후 Hadoop 2.0에서는 아키텍처에 큰 변화가 발생했다. MapReduce는 더 이상 유일한 실행 엔진이 아니게 되었고, Hadoop은 **YARN이라는 범용 분산 자원 관리 시스템 위에서 동작하는 플랫폼**으로 전환되었다. 이로 인해 Spark와 같은 다양한 분산 처리 엔진이 YARN 위에서 애플리케이션 레이어로 실행될 수 있는 구조가 마련되었다.

### HDFS: 분산 파일 시스템
---
`HDFS(Hadoop Distributed File System)`는 Hadoop의 핵심 구성 요소로, **대용량 데이터를 분산 환경에서 안정적으로 저장하기 위한 파일 시스템**이다. HDFS는 데이터를 일정 크기의 블록 단위로 나누어 저장하며, 기본 블록 크기는 128MB이다.

각 블록은 **복제(Replication) 방식**으로 여러 노드에 분산 저장된다. 기본적으로 하나의 블록은 **세 개의 서로 다른 노드에 중복 저장**되며, 이는 일부 노드에 장애가 발생하더라도 데이터 접근이 가능하도록 하는 `Fault Tolerance`를 보장한다. 블록들은 장애 상황을 고려한 배치 정책에 따라 저장된다.

Hadoop 2.0부터는 NameNode 이중화가 지원된다. Active와 Standby NameNode 구조를 통해 단일 장애 지점을 제거했으며, **두 NameNode는 공유된 Edit Log**를 통해 **메타데이터 상태를 동기화**한다. 이와 별도로 Secondary NameNode는 여전히 존재하며, 메타데이터 병합 작업을 담당한다.

![HDFS](/assets/img/contents/hdfs.png "HDFS Structure")

### MapReduce
---
`MapReduce`는 Hadoop 1.0에서 사용되던 **분산 컴퓨팅 모델**이다.

구조적으로는 **하나의 JobTracker와 다수의 TaskTracker로 구성**된다. JobTracker는 전체 작업을 관리하며, 작업을 여러 태스크로 분할해 TaskTracker에 분배한다. 각 TaskTracker는 할당된 태스크를 병렬로 처리한다.

MapReduce는 이름 그대로 Map과 Reduce 단계로 작업을 수행하며, 대규모 배치 처리에 적합하다. 다만 MapReduce만을 지원하는 구조로 인해 범용 **분산 컴퓨팅 시스템으로는 한계**가 있었고, 이러한 제약이 이후 Hadoop 2.0과 YARN, 그리고 Spark 등장으로 이어지게 된다.

![MapReduce](/assets/img/contents/mapreduce.png "MapReduce Structure")


## YARN의 동작 방식
---
### Hadoop 2.0: 범용 분산 컴퓨팅 프레임워크
---
Hadoop 2.0에서는 기존 Hadoop 1.0의 한계를 해결하기 위해 **YARN(Yet Another Resource Negotiator)**이 도입되었다. YARN은 세부적인 자원 관리가 가능한 **범용 분산 컴퓨팅 프레임워크**로, Hadoop을 단순한 MapReduce 플랫폼에서 다양한 분산 애플리케이션을 실행할 수 있는 환경으로 확장했다.

YARN의 주요 구성 요소는 다음과 같다.
클러스터 전체 자원을 관리하는 `Resource Manager(RM)`, 각 노드에서 자원 사용을 담당하는 `Node Manager(NM)`, 그리고 실제 작업 단위가 실행되는 `Container`가 있다. 각 애플리케이션마다 하나씩 할당되는 `Application Master(AM)`는 해당 애플리케이션의 실행을 총괄하며, 태스크 관리와 리소스 요청을 담당한다. Spark 역시 이러한 YARN 위에서 실행되는 대표적인 애플리케이션이다.

### YARN의 동작 방식
---
YARN에서 애플리케이션 실행은 다음과 같은 흐름으로 이루어진다.

먼저 **실행할 코드와 환경 정보가 Resource Manager에 제출**된다. 이때 실행에 필요한 파일들은 애플리케이션 ID에 해당하는 HDFS 디렉토리로 미리 복사된다.

Resource Manager는 Node Manager를 통해 Application Master를 실행한다. Application Master는 각 애플리케이션마다 하나씩 생성되며, **실행 로직을 관리하는 중심 역할**을 한다. 이후 Application Master는 **입력 데이터를 처리하는 데 필요한 리소스를 Resource Manager에 요청**하고, Resource Manager는 데이터 로컬리티를 고려해 적절한 컨테이너를 할당한다.

할당된 리소스는 Node Manager를 통해 컨테이너로 실행되며, 컨테이너 내부에서 실제 태스크가 수행된다. 이 과정에서 **필요한 파일들은 HDFS에서 해당 노드로 전달**된다. 각 태스크는 실행 상태를 주기적으로 Application Master에 heartbeat 형태로 보고하며, 태스크 실패나 응답 지연이 발생하면 다른 컨테이너에서 재실행된다.

![YARN](/assets/img/contents/yarn.png "YARN Structure")

### Hadoop 1.0과 Hadoop 2.0의 차이
---
Hadoop 1.0에서는 MapReduce가 자원 관리와 작업 실행을 모두 담당했지만, Hadoop 2.0에서는 **자원 관리 역할이 YARN으로 분리**되었다. 이를 통해 Hadoop은 MapReduce에 종속되지 않는 구조가 되었고, 다양한 분산 처리 엔진을 수용할 수 있는 플랫폼으로 발전했다.

![hadoop](/assets/img/contents/hadoopupdate.png "Hadoop v1.0 vs v2.0")

### Hadoop 3.0의 주요 특징
---
Hadoop 3.0은 기존 Hadoop 2.x 아키텍처를 기반으로 하면서, **자원 관리와 스토리지 확장성 측면에서 개선된 버전**이다. 핵심 변화는 YARN과 파일 시스템 영역에서 확인할 수 있다.

**YARN 2.0 기반 자원 관리**
Hadoop 3.0에서는 **YARN 2.0**이 사용된다. YARN은 애플리케이션들을 논리적인 그룹 단위로 묶어 관리할 수 있으며, 이러한 그룹을 Flow라고 부른다. 이를 통해 데이터 수집 파이프라인과 데이터 서빙 파이프라인처럼 성격이 다른 워크로드를 분리해 자원을 할당하고 관리할 수 있다.

또한 YARN의 타임라인 서버는 기본 스토리지로 **HBase**를 사용한다. 이는 **애플리케이션 실행 이력과 메트릭을 보다 안정적으로 저장하고 조회**하기 위한 개선 사항이다.

**파일 시스템 확장**
파일 시스템 측면에서도 확장성이 강화되었다. NameNode는 다수의 Standby NameNode를 지원함으로써 고가용성이 더욱 향상되었다.

또한 Hadoop 3.0은 기존 HDFS뿐만 아니라 **S3, Azure Storage, Azure Data Lake Storage 등 다양한 외부 스토리지 시스템을 지원**한다. 이를 통해 온프레미스 환경뿐 아니라 클라우드 기반 아키텍처에서도 유연하게 활용할 수 있다.

## 맵리듀스 프로그래밍 소개
---
MapReduce는 대규모 데이터를 처리하기 위한 **분산 프로그래밍 모델**로, 몇 가지 명확한 제약과 특징을 가진다. 먼저 MapReduce에서 다루는 데이터셋은 **Key-Value 쌍의 집합**이며, 한 번 생성된 데이터는 변경할 수 없는 **Immutable 구조**를 가진다.

데이터 조작은 오직 **Map과 Reduce 두 가지 오퍼레이션**을 통해서만 이루어진다. 이 두 오퍼레이션은 항상 하나의 쌍으로 연속 실행되며, 개발자는 각 단계에서 수행할 로직만 구현하면 된다. Map 작업의 결과는 시스템에 의해 자동으로 Reduce 단계로 전달된다.

이 과정에서 Map 결과를 Reduce로 전달하기 위해 **셔플링(Shuffle)** 단계가 발생하며, 네트워크를 통한 대량의 데이터 이동이 수반된다. 이는 MapReduce 성능에 큰 영향을 미치는 요소 중 하나이다.

### Hadoop 내에서의 Map과 Reduce의 역할
---
Map 단계는 입력 데이터를 변환하는 역할을 한다. 입력은 시스템에 의해 제공되며, 지정된 HDFS 파일로부터 Key–Value 형태로 전달된다. Map 함수는 `(k, v)` 형태의 입력을 받아, 새로운 Key–Value 쌍의 리스트 `[(k', v')...]`로 변환한다. 이 과정에서 입력을 그대로 출력할 수도 있고, 특정 조건에 따라 출력이 없을 수도 있다.

Reduce 단계는 Map 결과를 집계하는 역할을 한다. 시스템은 Map 출력 중 **같은 키를 가진 값들을 자동으로 묶어** Reduce 함수의 입력으로 전달한다. Reduce 함수는 `(k', [v1', v2', ...])` 형태의 입력을 받아 새로운 `(k'', v'')` 쌍으로 변환한다. 이는 SQL의 `GROUP BY` 연산과 매우 유사하며, 최종 결과는 HDFS에 저장된다.

![WordCount](/assets/img/contents/wordcount.png "WordCount")

### Shuffling과 Sorting
---
MapReduce에서 `Shuffling`은 Mapper의 출력 데이터를 **Reducer로 전달하는 과정**을 의미한다. 이 단계에서는 대량의 데이터가 네트워크를 통해 이동하게 되며, 전송되는 데이터 크기가 클수록 **네트워크 병목**이 발생하고 전체 처리 시간이 크게 증가한다.

Reducer는 전달받은 모든 Mapper의 출력을 **키 기준으로 Sorting한 뒤, Reduce 로직을 수행**한다. 이 과정 역시 추가적인 연산 비용을 발생시키며, 대규모 데이터 환경에서는 성능에 큰 영향을 미친다.

![Sorting](/assets/img/contents/sorting.png "Sorting")


### Data Skew 문제
---
MapReduce에서 자주 발생하는 문제 중 하나는 **Data Skew**이다. 이는 **각 태스크가 처리하는 데이터 양에 불균형이 존재하는 상황을 의미**한다. 병렬 처리 환경에서는 가장 느린 태스크가 전체 작업의 완료 시간을 결정하기 때문에, Data Skew가 발생하면 병렬 처리의 효과가 크게 감소한다.

특히 Group By나 Join 연산과 같이 특정 키에 데이터가 집중되는 경우, Reducer에 전달되는 데이터 크기에 큰 차이가 발생할 수 있다. 이는 메모리 부족 오류로 이어질 수 있으며, **빅데이터 시스템 전반에서 데이터 엔지니어가 반복적으로 마주치는 문제**이다.

### MapReduce 프로그래밍의 한계
---
MapReduce는 단순한 프로그래밍 모델을 제공하지만, **그만큼 생산성이 낮다는 한계**를 가진다. Map과 Reduce 두 가지 연산만 제공하기 때문에 복잡한 데이터 처리 로직을 표현하기 어렵고, 데이터 분포가 균등하지 않은 경우 성능 튜닝과 최적화도 쉽지 않다.

또한 MapReduce는 기본적으로 **배치 처리 중심의 시스템으로 설계**되어 있다. 이는 낮은 지연 시간보다는 **높은 처리량(Throughput)에 초점이 맞춰져 있어, 실시간성이나 인터랙티브 분석에는 적합하지 않다.**

### MapReduce 대안의 등장
---
이러한 한계를 극복하기 위해 보다 범용적인 대용량 데이터 처리 프레임워크들이 등장했다. 대표적으로 YARN 기반의 다양한 처리 엔진과 `Spark`가 있다.

또한 SQL 기반 분석에 대한 수요가 다시 증가하면서 `Hive`와 `Presto` 같은 엔진들이 등장했다. Hive는 **MapReduce 위에서 동작하며 대용량 ETL 처리에 적합**한 반면, Presto는 **메모리 기반 처리로 Low Latency 쿼리에 초점을 맞추며 Ad-hoc 분석에 적합**하다. AWS Athena는 Presto를 기반으로 한 대표적인 서비스이다.

## 하둡 설치와 맵리듀스 프로래밍 실습
---
### WordCount 실행 흐름
---
MapReduce의 대표적인 예제는 WordCount(단어 수 세기)이다. Hadoop은 예제 JAR을 기본 제공하며, 다음과 같이 실행할 수 있다.

- `bin/hadoop jar hadoop-*-examples.jar wordcount input output` (환경에 따라 `bin/hadoop`은 내부적으로 `bin/yarn`과 동일한 실행 엔트리로 동작한다.)

실행 이후에는 HDFS에 생성된 입력/출력 경로를 확인할 수 있다.

- `bin/hdfs dfs -ls input`
- `bin/hdfs dfs -ls output`

또한 실행 상태 및 결과는 Hadoop Web UI(Resource Manager)에서 잡 단위로 확인 가능하다. 이 과정을 통해 Map 태스크와 Reduce 태스크가 어떻게 분산 실행되는지, 리소스가 어떻게 할당되는지를 관찰할 수 있다.

### MapReduce의 구조적 문제
---
WordCount 같은 단순 작업에서는 MapReduce가 직관적이지만, 실무 관점에서는 한계가 분명하다. 우선 생산성이 낮다. 데이터 모델이 Key–Value로 고정되어 있고, 연산 역시 Map/Reduce 두 단계로 제한되기 때문에 복잡한 처리 로직을 작성하기가 어렵다.

또한 MapReduce는 **모든 입출력이 디스크(HDFS)를 중심으로 발생**한다. 이는 대규모 배치 처리에는 적합하지만, **반복 연산이나 인터랙티브 분석에는 비효율적**이다.

마지막으로 Shuffling 이후에는 **Data Skew가 발생하기 쉽고, Reducer 태스크 수 또한 개발자가 직접 지정**해야 한다. 데이터 분포가 균등하지 않은 경우 특정 Reducer로 데이터가 몰리면서 병목이 생기고, 태스크 수 설정에 따라 성능과 안정성이 크게 달라질 수 있다.

![MapReduce](/assets/img/contents/mapreduce_problem.png "MapReduce Problem")

## Spark 소개
---
Spark는 단순한 분산 처리 엔진을 넘어, **대용량 데이터 처리 전반을 아우르는 데이터 시스템으로 활용**된다. 대표적으로 배치 처리, 스트림 처리, 머신러닝 모델 빌딩 영역에서 널리 사용된다.

**활용 사례 개요**
Spark 데이터 시스템은 다음과 같은 시나리오에서 강점을 가진다:

- 대용량 데이터의 **배치 처리 및 스트림 처리**
- 머신러닝 모델 학습에 사용되는 **대규모 피처 데이터 처리**
- Spark ML을 활용한 **대규모 학습 데이터 기반 모델 훈련**

이러한 활용 사례들은 Spark의 메모리 기반 처리와 다양한 고수준 API 덕분에 효율적으로 구현할 수 있다.

**활용 사례 1: 대용량 비구조화 데이터 처리**
Spark는 로그 파일, 이벤트 데이터와 같은 **비구조화 또는 반구조화 데이터**를 처리하는 데 적합하다. 기존에는 Hive가 이러한 대용량 ETL/ELT 처리의 주요 수단이었으나, Spark는 **더 빠른 처리 성능과 유연한 API를 제공함으로써 이를 대체하거나 보완하는 역할**을 한다.

Spark SQL과 DataFrame API를 활용하면 대규모 데이터를 효율적으로 변환하고 정제할 수 있으며, 결과를 데이터 웨어하우스나 데이터 레이크로 적재하는 ETL 또는 ELT 파이프라인을 구성할 수 있다.

![Spark](/assets/img/contents/spark_ex1.png "Spark Example 01")

**활용 사례 2: 머신러닝 피처 처리**
머신러닝 모델 학습에서는 대량의 피처 데이터를 생성하고 가공하는 과정이 필수적이다. Spark는 배치와 스트림 환경 모두에서 **대규모 피처 엔지니어링**을 지원한다.

여러 데이터 소스를 조합해 피처를 생성하고, 이를 주기적으로 업데이트하거나 실시간으로 처리하는 작업에 Spark가 활용된다. 이러한 피처 데이터는 이후 Spark ML 또는 외부 머신러닝 플랫폼으로 전달되어 모델 학습에 사용된다.

![Spark](/assets/img/contents/spark_ex2.png "Spark Example 02")

## Spark 프로그램 실행 옵션
---
Spark 애플리케이션은 크게 `Driver`와 `Executor`로 구성된다. `Driver`는 **애플리케이션의 마스터 역할을 수행**하며, `Executor`는 **실제 연산을 수행하는 워커 역할을 담당**한다. YARN 환경에서는 Driver가 Application Master에 해당하고, Executor는 컨테이너(Container)로 실행된다.

![Spark Cluster](/assets/img/contents/spark_cluster.png "Spark Structure")

### Driver의 역할
---
Driver는 사용자가 작성한 Spark 코드를 실행하는 주체로, 실행 모드에 따라 위치가 달라진다. Client 모드에서는 클러스터 외부에서 실행되고, Cluster 모드에서는 클러스터 내부에서 실행된다.

Driver는 애플리케이션 실행에 필요한 리소스를 지정하며, 대표적으로 `--num-executors`, `--executor-cores`, `--executor-memory`와 같은 옵션을 통해 리소스를 설정한다. 또한 **SparkSession을 생성해 Spark 클러스터와 통신**하며, 클러스터 매니저(YARN의 경우 Resource Manager)와 Executor(YARN의 경우 Container)를 제어한다.

사용자 코드는 Driver에 의해 Spark 태스크 단위로 변환되고, 클러스터 전체에 분산 실행된다.

### Executor의 역할
---
Executor는 **실제 태스크를 실행하는 프로세스(JVM)**이다. Transformations과 Actions가 Executor에서 수행되며, 데이터 처리의 실질적인 작업이 이루어진다. YARN 환경에서는 각 Executor가 하나의 Container로 매핑된다.

### Spark 클러스터 매니저 옵션
---
Spark는 다양한 클러스터 매니저를 지원한다. 대표적으로 `local[n]`, `YARN`, `Kubernetes`, `Mesos`, `Standalone 모드`가 있다.

**local[n] 모드**
`local[n]` 모드는 개발 및 테스트 용도로 주로 사용된다. Spark Shell, IDE, 노트북 환경에서 실행할 때 적합하며, 하나의 JVM이 클러스터처럼 동작한다. 이 환경에서는 Driver와 하나의 Executor가 함께 실행된다.

여기서 `n`은 **사용할 CPU 코어 수를 의미**하며, Executor의 스레드 수로 사용된다. `local[*]`는 현재 머신에서 **사용 가능한 모든 코어를 사용**한다는 의미이다.

![Spark local](/assets/img/contents/localn.png "Spark local")

**YARN 모드**
YARN에서는 두 가지 실행 모드가 제공된다:

- `Client 모드`: Driver가 Spark 클러스터 외부에서 실행된다. YARN 기반 Spark 클러스터를 활용해 개발이나 테스트를 수행할 때 주로 사용된다.
- `Cluster 모드`: Driver가 Spark 클러스터 내부에서 실행되며, 하나의 Container 슬롯을 차지한다. 이 모드는 실제 프로덕션 환경에서 주로 사용된다.

**Spark 클러스터 매니저와 실행 모델 요약**

|클러스터 매니저|실행 모드(deployed mode)|프로그램 실행 방식|
|---|---|---|
|local[n]|Client|Spark Shell, IDE, 노트북|
|YARN|Client|Spark Shell, 노트북|
|YARN|Cluster|spark-submit|

## Spark 데이터 처리
---
**Spark 데이터 시스템 아키텍처**

![Spark Architecture](/assets/img/contents/spark_arch.png "Spark Architecture")

**데이터 병렬처리를 위한 전제 조건**
데이터 병렬 처리가 가능하려면, 가장 먼저 **데이터가 분산되어 있어야 한다.** Hadoop과 Spark 모두 데이터를 나누어 처리하는 구조를 전제로 한다.

Hadoop MapReduce에서는 데이터 처리의 최소 단위가 **HDFS 블록**이며, 기본 크기는 **128MB**이며, 이 값은 `hdfs-site.xml`의 `dfs.block.size` 설정에 의해 결정된다. 하나의 파일이 여러 블록으로 나뉘면, 각 블록마다 Map 태스크가 실행된다.

Spark에서는 이 개념을 **파티션(Partition)**이라 부른다. Spark 역시 기본 파티션 크기는 128MB이며, 파일을 읽을 때는 `spark.sql.files.maxPartitionBytes` 설정이 적용된다. 분산된 데이터는 파티션 단위로 메모리에 로드되고, 각 파티션이 Executor에 할당되어 병렬 처리된다.

![Parallel Processing](/assets/img/contents/parallel.png "Parallel Processing")

**Spark 데이터 처리 흐름**
Spark에서 DataFrame은 **여러 개의 작은 파티션으로 구성된 논리적 데이터 집합**이다. DataFrame은 생성 이후 수정할 수 없는 **Immutable 구조**를 가지며, 이는 분산 처리 환경에서 일관성과 안정성을 보장한다.

Spark의 데이터 처리는 입력 DataFrame을 시작으로, 원하는 결과가 나올 때까지 **연속적인 변환(Transformation)**을 수행하는 방식으로 이루어진다. 예를 들어 `filter`, `map`, `groupBy`, `join`, `sort`와 같은 연산들이 단계적으로 적용되며, 각 단계는 새로운 DataFrame을 생성한다.

![Spark Data Processing](/assets/img/contents/spark_data_proc.png "Spark Data Processing")

**셔플링(Shuffling)**
셔플링은 **파티션 간 데이터 이동이 필요한 경우에 발생**한다. 대표적인 예는 파티션 수를 명시적으로 변경하는 경우나, 시스템 내부적으로 데이터 재배치가 필요한 연산이다.

예를 들어 `groupBy`, `aggregation`, `sort`와 같은 연산은 동일한 키를 가진 데이터를 한 파티션으로 모아야 하므로, 네트워크를 통해 데이터가 이동하는 셔플링이 발생한다. 이 과정은 Spark 성능에 큰 영향을 미친다.

셔플 이후 생성되는 파티션 수는 `spark.sql.shuffle.partitions` 설정에 의해 결정되며, 기본값은 200이다. 이는 최대 파티션 수를 의미하며, 실제 파티션 수는 연산 방식에 따라 달라질 수 있다. Spark는 랜덤 파티셔닝, 해시 파티셔닝, 레인지 파티셔닝 등을 사용하며, 정렬 연산의 경우 주로 레인지 파티셔닝을 사용한다.

셔플링이 발생하는 시점에서는 **Data Skew가 발생할 가능성도 함께 존재**한다. 특정 키에 데이터가 집중될 경우 일부 파티션이 과도한 데이터를 처리하게 되어 전체 작업 성능을 저하시킬 수 있다.

**셔플링: hashing partition**
Spark에서 **Hash Partition**은 셔플링이 발생하는 대표적인 파티셔닝 방식 중 하나이다. 주로 **Aggregation 연산**에서 사용되며, 동일한 키를 가진 데이터가 반드시 같은 파티션으로 모이도록 보장한다.

Hash Partition은 레코드의 키 값을 해시 함수에 입력하고, 그 결과를 파티션 수로 나눈 값을 기준으로 파티션을 결정한다. 이 방식은 `groupBy`, `count`, `sum`과 같은 집계 연산에서 자연스럽게 사용된다.

Aggregation 연산에서는 동일 키에 대한 모든 레코드가 한 Reducer(또는 Spark의 경우 하나의 파티션)에서 처리되어야 하므로, 셔플 단계에서 **네트워크를 통한 데이터 재배치가 발생**한다. 이 과정은 데이터 규모가 클수록 비용이 커지며, Spark 성능 튜닝의 주요 고려 대상이 된다.

Hash Partition은 구현이 단순하고 균등 분산을 기대할 수 있지만, 키 분포가 불균형한 경우 **Data Skew**가 발생할 수 있다는 단점이 있다. 따라서 대규모 Aggregation 작업에서는 파티션 수 조정이나 키 설계에 대한 사전 고려가 필요하다.

![Hashing Partition](/assets/img/contents/hashing_partition.png "Hashing Partition")

**Data Skewness**
Data Skewness는 **분산 데이터 처리에서 데이터가 파티션 간에 균등하게 분포되지 않는 현상을 의미**한다. 데이터 파티셔닝은 병렬 처리를 가능하게 해 성능을 향상시키지만, 데이터 분포가 치우친 경우에는 오히려 성능 저하의 원인이 된다.

이 문제는 **주로 셔플링 이후에 발생**한다. `groupBy`, `join`, `aggregation`과 같은 연산에서 특정 키에 데이터가 집중되면, 일부 파티션이 과도한 데이터를 처리하게 된다. 그 결과 가장 느린 태스크가 전체 작업 시간을 결정하게 되어 병렬 처리의 이점이 크게 감소한다.

따라서 분산 처리 환경에서는 **셔플링을 최소화하는 것이 매우 중요**하며, 불가피하게 셔플이 발생하는 경우에는 **파티션 수 조정이나 키 설계와 같은 파티션 최적화 전략이 필요**하다. Data Skew를 인지하고 이를 완화하는 설계는 Spark 성능 튜닝의 핵심 요소 중 하나이다.

## Spark 데이터 구조: RDD, DataFrame, Dataset
---
Spark는 분산 환경에서 데이터를 처리하기 위해 **Immutable Distributed Data** 구조를 사용한다. 대표적인 데이터 구조로는 **RDD, DataFrame, Dataset**이 있으며, 이들은 모두 내부적으로 여러 개의 파티션으로 분할되어 병렬 처리된다.

2016년 이후 Spark에서는 DataFrame과 Dataset이 하나의 통합된 API로 정리되었으며, 사용 언어에 따라 노출되는 방식만 달라졌다.

![Spark DataStructures](/assets/img/contents/spark_datastructure.png "Spark DataStructures")

### RDD(Resilient Distributed Dataset)
---
RDD는 Spark의 가장 기본적인 데이터 구조로, **클러스터 내 여러 서버에 분산 저장된 로우레벨 데이터 집합을 의미**한다. 각 레코드는 독립적으로 존재하며, **스키마 정보가 없다는 것이 특징**이다. 이로 인해 구조화된 데이터와 비구조화된 데이터 모두를 처리할 수 있다.

RDD는 여러 개의 파티션으로 구성되며, `map`, `filter`, `flatMap`과 같은 **로우레벨 함수형 변환을 지원**한다. 일반적인 파이썬 컬렉션은 `parallelize` 함수를 통해 RDD로 변환할 수 있고, 반대로 `collect`를 사용하면 로컬 파이썬 데이터로 가져올 수 있다.

### DataFrame과 Dataset
---
DataFrame과 Dataset은 RDD 위에 구축된 **고수준 데이터 구조**로, RDD와 달리 명확한 필드(컬럼) 정보를 가진다. 개념적으로는 관계형 데이터베이스의 테이블이나 Pandas DataFrame과 매우 유사하다.

Dataset은 컬럼에 대한 타입 정보를 포함하며, 이는 컴파일 언어인 **Scala와 Java**에서만 사용할 수 있다. 반면 PySpark에서는 DataFrame API만 제공된다.

DataFrame은 HDFS, Hive, 외부 데이터베이스, 기존 RDD 등 **다양한 데이터 소스로부터 생성**할 수 있으며, Scala, Java, Python 등 여러 언어에서 동일한 추상화로 사용할 수 있다.

## 프로그램 구조
---
Spark 프로그램의 시작점은 **SparkSession을 생성**하는 것이다. SparkSession은 하나의 애플리케이션당 하나만 생성되는 Singleton 객체로, Spark 클러스터와의 모든 통신을 담당한다. 이 개념은 Spark 2.0부터 도입되었다.

SparkSession을 통해 Spark가 제공하는 다양한 기능을 사용할 수 있다. DataFrame과 SQL 처리뿐만 아니라 Streaming, ML API 역시 모두 SparkSession을 통해 접근한다. 환경 설정은 `config` 메서드를 사용해 지정할 수 있으며, RDD와 관련된 작업을 수행할 때는 **SparkSession 하위의 sparkContext 객체를 사용**한다.

**SparkSession이란?**
Spark 프로그램의 시작점은 S**parkSession을 생성하는 것**이다. SparkSession은 하나의 애플리케이션당 하나만 생성되는 **Singleton 객체**로, Spark 클러스터와의 모든 통신을 담당한다. 이 개념은 Spark 2.0부터 도입되었다.

SparkSession을 통해 Spark가 제공하는 다양한 기능을 사용할 수 있다. DataFrame과 SQL 처리뿐만 아니라 Streaming, ML API 역시 모두 SparkSession을 통해 접근한다. 환경 설정은 `config` 메서드를 사용해 지정할 수 있으며, RDD와 관련된 작업을 수행할 때는 SparkSession 하위의 **sparkContext 객체를 사용**한다.

**SparkSession 주요 환경 변수**
SparkSession을 생성할 때는 다양한 환경 변수를 설정할 수 있다. 대표적인 예는 다음과 같다.
- spark.executor.memory: Executor 당 메모리 크기 (기본값 1g)
- spark.executor.cores: Executor 당 CPU 코어 수 (YARN 기준 기본값 1)
- spark.driver.memory: Driver 메모리 크기 (기본값 1g)
- spark.sql.shuffle.partitions: 셔플 이후 생성되는 파티션 수 (기본값 최대 200)

실제로 사용할 수 있는 환경 변수는 매우 다양하며, 사용하는 **리소스 매니저(YARN, Kubernetes 등)**에 따라 설정 가능한 옵션도 달라진다.

```python
from pyspark.sql import SparkSession

# SparkSession은 싱글턴
spark = SparkSession.builder\
    .maaster("local[*]")\
    .appName('PySpark Tutorial')\
    .getOrCreate()

spark.stop
```

**SparkSession 환경 설정 방법**
Spark 환경 설정은 여러 방식으로 적용할 수 있다.

첫째, **환경 변수**를 통해 전역 설정이 가능하다.
둘째, `$SPARK_HOME/conf/spark-defaults.conf` 파일에 기본값을 정의할 수 있다.
셋째, `spark-submit` 명령 실행 시 **커맨드라인 파라미터로 설정**할 수 있으며, 이는 실행 단위의 설정에 유용하다.
마지막으로 SparkSession을 생성할 때 코드 레벨에서 직접 설정할 수도 있다.

이러한 다양한 설정 방식을 통해 개발 환경과 운영 환경에 맞는 Spark 실행 구성을 유연하게 관리할 수 있다.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder\
    .maaster("local[*]")\
    .appName('PySpark Tutorial')\
    .config("spark.some.config.option1", "some-value")\
    .config("spark.some.config.option2", "some-value")\
    .getOrCreate()
```

```python
from pyspark.sql import SparkSession
from pyspark import SparkConf

conf = SparkConf()
conf.set("spark.app.name", "PySpark Tutorial")
conf.set("spark.master", "local[*]")

# SparkSession은 싱글턴
spark = SparkSession.builder\
    .config(conf=conf) \
    .getOrCreate()
```

**Spark 데이터 처리의 전체적인 흐름**
Spark 애플리케이션은 일정한 처리 흐름을 따른다. 가장 먼저 **SparkSession을 생성**하며, 이를 통해 Spark 클러스터와 통신을 시작한다. SparkSession은 애플리케이션의 진입점 역할을 한다.

다음으로 **입력 데이터를 로딩**한다. 이 단계에서 데이터는 DataFrame 형태로 로드되며, 이후 모든 처리는 이 DataFrame을 중심으로 이루어진다.

데이터 로딩 이후에는 **데이터 조작 작업**이 수행된다. 이 과정은 Pandas와 매우 유사하며, DataFrame API나 Spark SQL을 사용해 `filter`, `groupBy`, `join` 등의 연산을 적용한다. Spark의 데이터 구조는 Immutable하기 때문에, 각 연산은 기존 DataFrame을 수정하는 것이 아니라 **새로운 DataFrame을 생성**한다. 원하는 결과가 나올 때까지 이러한 변환을 반복한다.

마지막으로 **최종 결과를 저장**한다. 결과 데이터는 파일 시스템이나 데이터베이스 등 다양한 저장소로 출력될 수 있다.

**SparkSession이 지원하는 데이터 소스**
SparkSession은 다양한 데이터 소스를 통합적으로 지원한다. 데이터 로딩 시에는 `spark.read`(DataFrameReader)를 사용해 DataFrame으로 불러오고, 저장 시에는 `DataFrame.write`(DataFrameWriter)를 사용한다.

Spark에서 자주 사용되는 데이터 소스는 다음과 같다:

- **HDFS 파일**: CSV, JSON, Parquet, ORC, Text, Avro 등의 포맷을 지원한다. 이 중 Parquet, ORC, Avro와 같은 컬럼 기반 포맷은 대규모 데이터 처리에 특히 유리하다.
- **Hive 테이블**: 메타스토어를 통해 Hive 테이블을 직접 DataFrame으로 로드할 수 있다.
- **JDBC 기반 관계형 데이터베이스**: MySQL, PostgreSQL 등 전통적인 RDBMS와 연동 가능하다.
- **클라우드 기반 데이터 시스템**: S3, GCS, Azure Storage 등과 같은 클라우드 스토리지를 지원한다.
- **스트리밍 시스템**: Kafka와 같은 스트리밍 데이터 소스를 통해 실시간 데이터 처리도 가능하다.

## Spark 데이터베이스
---
**카탈로그(Catalog)**
Spark에서는 **카탈로그(Catalog)**를 통해 테이블과 뷰에 대한 **메타데이터를 관리**한다. 기본적으로 Spark는 메모리 기반 카탈로그를 제공하며, 이는 세션 단위로 유지되어 SparkSession이 종료되면 함께 사라진다.

보다 영속적인 메타데이터 관리를 위해 **Spark는 Hive와 호환되는 카탈로그**를 지원한다. 이 경우 메타데이터는 외부 메타스토어에 저장되며, 세션 종료 이후에도 유지된다.

**데이터베이스와 테이블 관리 방식**
Spark에서 테이블은 **데이터베이스(Database)**라 불리는 논리적 단위로 관리된다. 데이터베이스는 파일 시스템 상의 폴더와 유사한 역할을 하며, 테이블과 뷰를 계층적으로 관리하는 2단계 구조를 가진다.

**메모리 기반 테이블과 뷰**
메모리 기반 테이블과 뷰는 **임시 테이블**로, 주로 세션 내에서만 사용된다. 앞서 사용한 임시 뷰들이 이에 해당하며, SparkSession이 종료되면 자동으로 제거된다. 빠른 실험이나 중간 결과를 확인하는 용도로 적합하다.

**스토리지 기반 테이블**
스토리지 기반 테이블은 실제 데이터가 파일 시스템에 저장되는 테이블이다. 기본적으로 **HDFS와 Parquet 포맷을 사용**하며, 메타데이터는 Hive와 호환되는 메타스토어를 통해 관리된다.

스토리지 기반 테이블은 Hive와 동일하게 두 가지 유형으로 구분된다.
**Managed Table은 Spark가 데이터와 메타데이터를 모두 관리하는 테이블**이며, 테이블 삭제 시 실제 데이터도 함께 제거된다.
반면 **Unmanaged(External) Table**은 Spark가 메타데이터만 관리하며, 실제 데이터는 외부에서 관리된다. 이 경우 테이블을 삭제해도 데이터 파일은 유지된다.

![Spark Database](/assets/img/contents/spark_db.png "Spark Database")


## Spark 파일 포맷
---
**Parquet: Spark의 기본 파일 포맷**
Parquet는 Spark에서 기본적으로 사용되는 **컬럼 지향(Columnar) 파일 포맷**이다. 트위터와 클라우데라가 공동으로 개발했으며, Hadoop 생태계 전반에서 표준 포맷으로 자리 잡았다. Parquet는 컬럼 단위 저장 방식을 통해 디스크 I/O를 최소화하고, 압축 효율을 높이며, 대규모 분석 쿼리에 최적화된 성능을 제공한다.

**Spark 실행 단위: Job, Stage, Task**
Spark에서 코드 실행은 **Action을 기점으로 실제 수행**된다. 하나의 Action은 하나의 Job을 생성하며, Job은 하나 이상의 Stage로 구성된다.

Stage는 **셔플링이 발생하는 지점을 기준으로 분리**된다. 즉, 셔플이 없는 연산들은 하나의 Stage로 묶이고, 셔플이 발생하면 새로운 Stage가 생성된다. 각 Stage는 DAG 형태로 구성된 여러 Task를 포함하며, 이 Task들은 병렬로 실행된다.

Task는 **Spark 실행의 가장 작은 단위**로, 각 Executor에 의해 실제로 수행된다. 전체 실행 성능은 Task 수, 파티션 수, 그리고 셔플 구조에 크게 영향을 받는다.

## Execution Plan
---
**Bucketing과 File System Partitioning**
Spark에서는 데이터를 저장할 때 이후의 반복 처리 성능을 고려한 저장 최적화 전략을 사용할 수 있다. 대표적인 방법이 Bucketing과 File System Partitioning이며, 두 방식 모두 **Hive 메타스토어를 사용하는 테이블(saveAsTable)**에서 활용된다.

```python
spark.read.option("header", True). \
    csv(“test.csv”). \
    where("gender <> 'F'"). \
    select("name", "gender"). \
    groupby("gender"). \
    count(). \
    show()
```

## Bucketing과 Partitioning
---
**Bucketing**
Bucketing은 DataFrame을 특정 컬럼(ID 등)을 기준으로 **해시 분할하여 테이블로 저장하는 방식**이다. 주로 Aggregation, Window 함수, Join에서 자주 사용되는 컬럼이 있을 때 효과적이다.

버킷 수와 기준 컬럼을 지정해 데이터를 저장하면, 이후 동일한 조건의 연산에서 데이터 재분배 비용이 줄어들어 반복 처리 성능이 향상된다. Spark에서는 `DataFrameWriter.bucketBy()` 함수를 사용해 Bucketing을 적용한다. 이 방식은 데이터 특성을 잘 알고 있는 경우에 특히 유용하다.

**File System Partitioning**
File System Partitioning은 Hive에서 널리 사용되던 방식으로, 특정 컬럼 값을 기준으로 **디렉토리 구조를 나누어 데이터를 저장**한다. 이때 사용되는 컬럼을 Partition Key라고 부른다.

이 방식은 조건절에 Partition Key가 포함될 경우 불필요한 데이터 스캔을 줄여주며, 대규모 테이블 조회 성능을 크게 개선한다


**Partitioning의 예와 장점**
예를 들어 매우 큰 로그 데이터를 자주 조회하는 상황에서, 데이터 생성 시간을 기준으로 데이터를 읽는 경우가 많다면 **연도–월–일과 같은 디렉토리 구조로 데이터를 저장하는 것이 효과적**이다. 실제로 많은 로그 데이터는 이미 이러한 형태로 수집·저장된다.

이와 같은 파티셔닝 구조를 사용하면, 조회 시 조건에 해당하는 폴더만 스캔하게 되어 **불필요한 데이터 읽기 비용이 크게 줄어든다.** 그 결과 쿼리 성능이 개선되며, 데이터 스캔 자체가 발생하지 않는 경우도 있다. 또한 기간 단위로 데이터를 관리할 수 있어 **Retention Policy 적용**과 같은 운영 작업도 수월해진다.

**Partitioning 적용 시 주의사항**
Spark에서는 `DataFrameWriter.partitionBy()`를 사용해 File System Partitioning을 적용한다. 다만 Partition Key를 잘못 선택할 경우, 파티션 수가 과도하게 늘어나 **매우 많은 작은 파일들이 생성**될 수 있다. 이는 메타데이터 관리 부담과 성능 저하로 이어질 수 있으므로, 파티션 키는 데이터 접근 패턴과 카디널리티를 고려해 신중하게 선택해야 한다.