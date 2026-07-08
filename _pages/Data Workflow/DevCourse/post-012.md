---
title: "[6기] 데브코스 DE WIL 12 | Kafka & Spark Streaming 기반 스트리밍 처리"
tags:
    - DevCourse
    - Data Engineering
    - Kafka
date: "2025-06-19"
thumbnail: "/assets/img/thumbnail/devcourse.png"
bookmark: true
---

## 이번 주 학습 목표
---
- 이벤트 중심 아키텍처의 관점에서 **실시간 데이터 처리 흐름과 Kafka 기반 스트리밍 파이프라인의 구조를 이해**한다.
- **Kafka의 핵심 구성 요소(Topic, Partition, Broker, Consumer Group, Connect 등)와 메시지 처리 보장 방식을 이해**하고, 실무 환경에서의 설계 선택 기준을 정리한다.
- Spark Structured Streaming의 **마이크로 배치 처리 모델과 Source–Sink 구조를 이해하고, Kafka와 연계한 실시간 데이터 처리 파이프라인을 설계**할 수 있다.

## 데이터 처리의 발전 단계
---
데이터 파이프라인의 데이터 처리는 보통 세 가지 단계로 구성된다. `데이터 수집(Data Collection)`, `데이터 저장(Data Storage)`, `데이터 처리(Data Processing)`이다.

먼저, 데이터 수집 단계에서는 서비스나 시스템에서 발생하는 다양한 데이터를 모은다. **사용자 행동 로그, 서버 로그, 결제 데이터, 외부 API** 등이 여기에 해당된다. 이 단계에서는 얼마나 빠르고 안정적으로 데이터를 수집할 수 있는지가 중요하다.

수집된 데이터는 이후 데이터 저장(Data Storage) 단계로 넘어간다. 데이터 성격과 사용 목적에 따라 저장소는 달라진다. **트랜잭션 처리가 중요한 데이터는 데이터베이스에 저장**되고, **대규모 분석을 위한 데이터는 데이터 레이크나 데이터 웨어하우스에 적재**된다.

**시대별 데이터 저장 시스템의 변천**

|1980년대|2000년대 후반|2010년 중반|2021년|
|---|---|---|---|
|Data Warhouse(Top-down)|Data Lake(Bottom-up)|Cloud Data Platform Messing Queue(Kafka/Kinesis)|Data Mesh|
|중앙 시스템|중앙 시스템|중앙 시스템|분산 시스템|

마지막으로 데이터 처리 단계에서는 저장된 데이터를 정제하고 변환하며, 필요한 형태로 가공한다. 이 과정을 통해 서비스 효율을 높이거나, 데이터 기반의 의사결정을 보다 과학적으로 수행할 수 있게 된다. 데이터 분석, 추천 시스템, 모니터링 대시보드 등 대부분의  데이터 활용 사례는 이 단계에서 만들어진다.

**데이터 처리 방식의 고도화**
초기 시스템은 대부분 배치 처리로 시작된다. 배치 처리는 일정 주기로 데이터를 모아 한 번에 처리하며, 한 번에 처리할 수 있는 데이터의 양이 중요하다. 이러한 방식은 대규모 분석이나 리포트 생성에 적합하다.

서비스가 성장함에 따라 데이터의 즉각적인 활용이 요구되면서 **실시간 처리 또는 준실시간** 처리가 필요해진다. 실시간 처리는 데이터 발생 즉시 처리하는 방식을, 준실시간 처리는 약간의 지연을 허용하는 처리 방식이다. 또한 하나의 데이터가 여러 시스템에서 동시에 사용되는 경우가 늘어나면서, 다수의 데이터 소비자를 고려한 구조가 중요해진다.

**처리량(Throught) vs 지연시간(Latency)**
데이터 처리 성능을 이해하기 위해서 `처리량`과 `지연시간`의 차이를 명확히 이해한다. 

`처리량`은 단위 시간당 처리할 수 있는 데이터 양으로, 배치 시스템에서 중요한 지표이다. 반면 `지연시간`은 데이터가 처리되기까지 걸리는 시간으로, 실시간 시스템에서 더 중요하다.

해당 두 개념은 대개 **트레이드오프(trade-off) 관계**에 있으며, 이를 함께 설명하는 개념으로 **대역폭(Bandwidth)**이 있다. 대역폭은 처리량과 지연시간의 곱으로 표현된다.

**SLA(Service Level Agreement)**
SLA는 **서비스 제공자와 사용자 간에 합의된 서비스 품질 기준**을 의미한다. 이 계약에는 서비스의 성능, 가용성, 신뢰성 등에 대한 최소 기준이 포함되며, 통신, 클라우드 컴퓨팅 등 다양한 산업에서 널리 사용된다.

SLA는 외부 고객뿐만 아니라 **사내 시스템 간에도 정의**된다. 이 경우 주로 지연시간(Latency)이나 업타입이 기준이 된다. 

데이터 시스템에서는 여기에 더해 **데이터의 시의성(Freshness)**이 중요한 SLA 지표가 된다. 데이터가 얼마나 최신 상태로 제공되는지가 분석과 의사결정의 품질에 직접적인 영향을 주기 때문이다.

### 배치 처리의 특징과 구조
---
배치 처리는 데이터를 **주기적으로 모아서 한 번이 이동하거나 처리하는 방식**이다. 이 방식에서는 처리량(Thoughput)이 가장 중요한 성능 지표이며, 처리 주기는 보통 분, 시간, 일 단위로 설정된다.

배치 처리 시스템은 일반적으로 분산 파일 시스템과 분산 처리 엔진을 중심으로 구성된다. 데이터는 HDFS나 S3와 같은 분산 스토리지에 저장되고, `MapReduce`, `Hive`, `Presto`, `Spark(DataFrame, SQL)` 등을 통해 처리된다. 

이러한 작업을 정해진 주기로 안정적으로 실행하기 위해서 **Airflow와 같은 워크플로 스케줄러가 사용**되는 경우가 많다.

![Batch Processing](/assets/img/contents/batch_proc.png "Batch Processing")

### 실시간 데이터 처리로의 발전
---
배치 처리 이후의 고도화 단계가 바로 **실시간 데이터 처리**이다. 실시간 처리는 시스템 복잡도가 증가하지만, 초 단위로 지속적으로 발생하는 데이터를 즉시 처리할 수 있다는 장점이 있다.

이때 처리되는 데이터로는 되게 **이벤트(Event)**라고 부르며, 이벤트의 가장 큰 특징은 생성 이후이 값이 변경되지 않는 **불변성(Immutable)**이다. 이러한 이벤트들이 지속적으로 발생하며 흐름을 이루는 것을 **이벤트 스트림(Event Stream)**이라고 한다.

![Event Stream](/assets/img/contents/event.png "Event Stream")

![Realtime Processing](/assets/img/contents/realtime_proc.png "Realtime Processing")

**실시간 처리 시스템의 구성 요소**
배치 처리만으로는 이러한 이벤트 스트림을 효과적으로 다루기 어렵기 때문에, 실시간 처리 환경에서는 새로운 유형의 시스템들이 필요해진다.

먼저 이벤트 데이터를 안정적으로 저장하고 전달하기 위해 **메시지 큐 시스템**이 사용된다. `Kafka`, `Kinesis`, `Pub/Sub` 등이 대표적인 예이며, 이들은 대량의 이벤트 스트림을 안정적으로 보관하고 전달하는 역할을 한다.

그 위에서 **이벤트를 실제로 처리하기 위한 스트리밍 처리 엔진이 동작**한다. `Spark Streaming`, `Samza`, `Flink`와 같은 시스템은 이벤트 스트림을 읽어 실시간 집계, 변환, 계산을 수행한다. 처리된 결과를 다시 저장되거나 `Druid`와 같은 실시간 분석 및 대시보드 시스템으로 전달되어 즉각적인 분석에 활용된다.

**실시간 처리 시스템의 구조**
실시간 데이터 처리 아키텍처는 일반적으로 **Producer -> Message Queue -> Consumer** 구조를 따른다.

먼저, `Producer(또는 Publisher)`는 **이벤트를 생성하는 주체**로, 애플리케이션이나 서비스가 여기에 해당한다. 생성된 이벤트는 `Kafka`나 `Kinesis`와 같은 메시지 큐 시스템에 저장된다. 이때 **데이터는 스트림 단위로 관리**되며, Kafka에서는 이를 `토픽(Topic)`이라고 부른다. 각 스트림에는 데이터 보유 기간이 설정되어 있어, 일정 시간이 지나면 자동으로 삭제된다.

Consumer(또는 Subscriber)는 메시지 큐로부터 이벤트를 읽어 처리하는 주체다. 각 Consumer는 자신만의 처리 위치를 관리하며, 하나의 스트림을 여러 Consumer가 동시에 읽는 것도 가능하다. 이를 통해 동일한 이벤트 데이터를 여러 목적의 시스템에서 동시에 활용할 수 있다.

![Data Realtime Processing](/assets/img/contents/data_realtime_proc.png "Data Realtime Processing")

### 데이터 실시간 처리의 장/단점
---
서비스가 고도화될수록 데이터는 단순한 분석의 대상이 아닌, **즉시 반응해야 하는 신호**가 된다. 이러한 요구를 충족하기 위해서 등장한 것이 데이터 실시간 처리이며, 이는 분명한 장점과 단점을 가지고 있다.

**데이터 실시간 처리의 장점**
실시간 처리의 가장 큰 장점은 **즉각적인 인사이트 확보**다. 데이터가 생성되는 순간 바로 분석이 가능하기 때문에, 서비스 운영자는 현재 상황을 빠르게 파악할 수 있다. 이는 운영 효율성 향상으로 이어지며, 장애나 사고와 같은 이벤트에 대해서도 신속하게 대응할 수 있게 한다.

또한 실시간 데이터 처리는 사용자 행동을 즉시 반영할 수 있어, **개인화된 사용자 경험**을 더욱 정교하게 만든다. 추천 시스템, 실시간 알림, 동적 UI 변화 등이 대표적인 사례다.
이와 함께 IoT 및 센서 데이터 활용, 사기 탐지 및 보안 시스템, 실시간 협업과 커뮤니케이션 기능 등에서도 실시간 처리는 중요한 역할을 한다.

**데이터 실시간 처리의 단점**
반면 실시간 처리는 **시스템 전반의 복잡도를 크게 증가**시킨다. 배치 시스템은 주기적으로 동작하며 대부분 사용자에게 직접 노출되지 않지만, 실시간 시스템은 실제 사용자 경험과 밀접하게 연결되는 경우가 많다. 이로 인해 장애 발생 시 즉각적인 대응이 필수적이며, 시스템 안정성이 매우 중요해진다.

예를 들어 배치 기반 추천 시스템과 달리 실시간 추천 시스템은 장애가 곧바로 서비스 품질 저하로 이어진다. 이 시점부터 데이터 처리는 단순한 데이터 엔지니어링을 넘어 **DevOps 영역**과 맞닿게 된다.

또한 운영 비용 역시 증가한다. 배치 처리에서는 오류가 발생하더라도 데이터 유실 가능성이 상대적으로 낮지만, 실시간 처리에서는 데이터 유실 위험이 커진다. 따라서 지속적인 백업과 모니터링이 필요하며, 이는 곧 인프라 및 운영 비용 증가로 이어진다.

**Realitme vs Semi-Realtime**
실시간 데이터 처리는 다시 **Realtime과 Semi-Realtime**으로 나눌 수 있다.

`Realtime` 처리는 매우 짧은 지연시간을 목표로 하며, 연속적인 데이터 스트림을 기반으로 동작한다. 이벤트 중심 아키텍처를 사용하여, 데이터가 수신되는 즉시 작업이나 계산이 트리거된다. 이러한 구조는 데이터 변화에 동적으로 반응하며 실시간 분석, 모니터링, 의사결정을 가능하게 한다.

반면, `Semi-Realtime` 처리는 완전한 즉시성을 약간 포기하고, 합리적인 지연시간을 선택한다. 보통 마이크로 배치(Micro-batch) 방식으로 동작하며, 배치 처리와 유사한 구조를 가진다. 이는 처리 용량과 리소스 활용 효율을 높이기 위한 선택으로, 적시성과 효율성 사이의 균형을 추구한다. 데이터는 짧은 주기로 업데이트되며, 많은 실무 환경에서 현실적인 대안으로 사용된다.

## 실시간 데이터 종류와 사용 사례
---
실시간 데이터 처리를 이해하기 위해 가장 먼저 받아들여야 할 관점은 **"이벤트는 어디에나 존재한다(Events are Everywhere)"**는 사실이다. 현대의 서비스와 비즈니스 환경에서는 대부분의 변화와 행동이 이벤트의 형태로 발생하며, 데이터 실시간 처리는 이러한 이벤트를 빠짐없이 포착하고 활용하는 데서 시작된다.

**Case 01: 온라인 서비스에서의 이벤트**
온라인 서비스에서는 사용자의 거의 모든 행동이 이벤트로 기록될 수 있다. 대표적인 예가 **퍼널 데이터(Funnel Data)**다. 상품 노출, 클릭, 구매와 같은 일련의 사용자 행동은 클릭 스트림 형태로 수집되며, 회원가입 역시 버튼 클릭부터 정보 입력, 최종 등록까지 여러 이벤트의 연속으로 표현된다.

또한 페이지 뷰와 성능 데이터 역시 중요한 이벤트다. 페이지별 렌더링 시간을 기록해두면, 서비스 장애나 성능 저하가 발생했을 때 원인을 빠르게 추적할 수 있다. 이때 데스크탑과 모바일 등 **디바이스 타입별로 이벤트를 구분**하면 분석의 정확도가 높아진다. 페이지 에러가 발생하는 경우에도 에러 이벤트를 별도로 수집하여 문제를 체계적으로 관리할 수 있다.

이 밖에도 사용자 등록, 로그인, 방문자 발생 등 기본적인 사용자 행동 모두 이벤트에 해당한다. 이러한 데이터가 늘어나면서, **이벤트 데이터의 모델을 정의하고 정확히 수집하는 작업의 중요성**이 점점 커지고 있다. 데이터가 올바르게 수집되어야 이후 저장과 소비가 가능하기 때문에, 이벤트 수집만을 전담하는 팀이 생기기도 한다.

**Case 02: 리테일 비즈니스에서의 이벤트**
리테일 환경에서도 이벤트는 핵심적인 역할을 한다. 재고 추가나 품절과 같은 재고 수준 변화는 모두 이벤트로 기록되며, 이를 통해 실시간 재고 관리가 가능해진다.

**Case 03: IoT 환경에서의 이벤트**
IoT 환경에서는 이벤트의 비중이 더욱 커진다. 센서에서 수집되는 온도, 습도, 압력과 같은 측정값은 대표적인 이벤트 데이터다. 여기에 더해 장치의 온라인/오프라인 상태나 배터리 잔량과 같은 장치 상태 변화 역시 이벤트로 관리된다.

### 이벤트 데이터가 필요한 주요 유스 케이스
---
이벤트 데이터는 다양한 실시간 유스 케이스의 기반이 된다. 실시간 리포팅 영역에서는 `A/B 테스트 분석`, `마케팅 캠페인 대시보드`, `인프라 모니터링` 등이 대표적이다. 또한 실시간 알림 영역에서는 사기 탐지, 실시간 입찰, 원격 환자 모니터링과 같은 사례가 존재한다.

나아가 이벤트 데이터는 실시간 예측에도 활용된다. 머신러닝 모델을 통해 사용자 행동을 즉시 반영한 **개인화 추천 시스템**은 이벤트 기반 데이터 처리의 대표적인 결과물이라고 볼 수 있다.

## 실시간 데이터 처리 단계
---
실시간 데이터 처리는 단일 기술이나 시스템으로 완성되지 않는다. 이벤트가 정의되는 시점부터 전송, 처리 그리고 운영까지 **여러 단계를 체계적으로 설계**해야 실시간 시스템을 구축할 수 있다.

실시간 데이터 처리 과정은 크게 네 단계로 나눌 수 있다. 먼저 **이벤트 데이터 모델을 결정**하고, 이를 **전송 및 저장**한 뒤, 실제로 **데이터를 처리**한다. 마지막으로 전체 과정에서 발생할 수 있는 **관리 이슈를 모니터링하고 해결**하는 단계가 필요하다. 

이 네 단계는 서로 강하게 연결되어 있으며, 앞단의 선택이 뒤 단계의 복잡도와 안정성에 직접적인 영향을 미친다.

### **1단계: 이벤트 데이터 모델 결정**
---
이벤트 데이터 모델을 설계할 때 가장 최소한의 필요 요소는 **Primary Key와 Timestamp**이다. 이를 통해 이벤트의 식별과 시간 순서가 보장된다.

유스케이스에 따라 사용자 정보가 포함될 수도 있고, 이벤트 자체에 대한 세부 정보가 필요할 수도 있다. 예시로 클릭 이벤트의 경우, 어떤 페이지에서 어떤 버튼이 클릭되었는가와 같은 정보가 이벤트 데이터에 포함된다.

이 단계에서의 모델 설계는 이후 저장 방식과 처리 방식까지 좌우하게 되므로 중요하다고 볼 수 있다.

### **2단계: 이벤트 데이터 전송/저장**
---
이벤트 데이터를 전닿하고 저장하는 방식은 크게 **Point-to-Point** 방식과 **Message Queue** 방식으로 나뉜다.

`Point-to-Point` 방식은 Producer와 Consumer가 직접 연결되는 구조이다. 처리량도 중요하지만, 무엇보다 **지연 시간이 매우 중요한 시스템에서 사용**된다. 많은 API 레이어가 이와 같은 방식으로 동작한다. 다만 Consumer가 여러 개인 경우 동일한 데이터를 반복해서 전송해야 하며, 확장성이 떨어진다는 단점이 있다.

![Point-to-Point](/assets/img/contents/point_to_point.png "Point-to-Point")

`Nessage Queue` 방식은 Producer와 Consumer 사이에 Kafka와 같은 메시지 큐를 두는 구조이다. 중간 저장소를 통해 생산자와 소비자가 분리되며, 시스템 간 결합도가 낮아진다. 이 방식은 실시간 데이터 처리에서 가장 일반적으로 사용된다.

![Message Queue](/assets/img/contents/message_queue.png "Message Queue")

**(번외) Backpressure(배압) 이슈**
실시간 처리 시스템에서 반드시 고려해야 할 문제가 `Backpressure`이다. 이벤트 데이터는 일반적으로 일정한 속도로 생성되지만, 특정 상황에서는 데이터 생산량이 급격히 증가할 수 있다.

문제는 Consumer가 이 속도를 따라가지 못할 때 발생한다. **처리 지연이 누적되면 메모리 사용량이 증가하고, 이는 잠재적인 시스템 장애**로 이어질 수 있다. 이를 Backpressure 이슈라고 부른다.

Backpressure를 완화하는 대표적인 방법 중 하나가 메시지 큐를 중간에 두는 것이다. 메시지 큐는 데이터 폭증 상황에서 완충 역할을 하지만, 이 문제는 완전히 제거할 수는 없다. Point-to-Point 구조에서도 Consumer 쪽에 작은 버퍼가 존재하지만, 버퍼 크기는 제한적이기 때문에 결국 오버플로우 문제가 발생할 수도 있다.

### **3단계: 이벤트 데이터 처리**
---
이벤트 데이터를 어떻게 처리할지는 앞서 선택한 전송·저장 모델과 활용 목적에 따라 결정된다.

`Point-to-Point` 방식에서는 Consumer의 부담이 크며, **데이터가 들어오는 즉시 처리**되어야 한다. 이로 인해 Backpressure에 취약하고 데이터 유실 가능성도 상대적으로 높다. 일반적으로 Low Throughput, Low Latency 특성을 가진다.

`Messaging Queue`를 사용하는 경우에는 보통 **마이크로 배치(Micro-batch) 형태로 아주 짧은 주기 동안 데이터를 모아 처리**한다. `Spark Streaming`이 대표적인 예다. 이 방식은 다수의 Consumer를 쉽게 구성할 수 있고, Point-to-Point 방식에 비해 운영이 훨씬 수월하다는 장점이 있다.

### **4단계: 이벤트 데이터 관리 이슈 모니터링 및 해결**
---
이벤트 데이터 처리는 **구현보다도 운영 단계에서의 안정성 관리**가 핵심이다. 실시간 시스템은 지속적으로 동작하며 사용자 경험과 직접 연결되기 때문에, 작은 문제도 빠르게 장애로 이어질 수 있다.

가장 중요한 모니터링 대상은 **데이터 지연(Lag)**이다. Producer에서 생성된 이벤트가 Consumer에서 처리되기까지의 지연이 지속적으로 증가한다면, Backpressure나 처리 성능 저하를 의심해야 한다. 메시지 큐 기반 시스템에서는 Consumer Lag이 대표적인 지표로 사용된다.

또한 이벤트가 정상적으로 수집·저장·처리되고 있는지 확인하여 **데이터 유실 가능성을 최소화**해야 한다. 이를 위해 오프셋 관리, 재처리 가능 여부, 데이터 보존 기간 설정이 중요하다.

문제가 발생하면 Consumer 확장, 처리 로직 최적화, 파티션 조정 등 구조적인 개선을 통해 대응한다. 경우에 따라서는 완전한 실시간 처리 대신 Semi-Realtime 방식이 더 적절한 선택이 될 수도 있다.

## Kafka 소개
---
Kafka는 **실시간 데이터 처리를 위해 설계된 오픈소스 분산 스트리밍 플랫폼**이다. 단순한 메시지 큐를 넘어, 데이터를 일정 기간 저장하고 다시 읽을 수 있는 **분산 커밋 로그(Distibuted Commit Log)** 개념을 기반으로 한다. 이로 인해 Kafka는 대규모 실시간 데이터 파이프라인의 중심 역할을 수행한다.

Kafka는 Publish-Subscribe 모델을 따르는 메시징 시스템으로, Producer와 Consumer가 명확히 분리되어 있다. 이를 통해 시스템은 높은 처리량과 지연시간을 동시에 만족할 수 있으며, 실시간 데이터 처리에 최적화된 구조를 가진다.

Kafka는 분산 아키텍처를 기반으로 하며, 서버를 추가하는 **Scale-Out 방식**으로 확장된다. 이때 각 서버는 브로커(Broker)라고 불리며, 브로커를 추가함으로써 자연스럽게 처리량과 저장 용량을 확장할 수 있다.

Kafka는 메시지를 일정 기간 동안 저장한다. 이 보유 기간(retention period) 동안 데이터는 삭제되지 않으며, 기본 설정은 약 일주일이다. 이 특성 덕분에 Consumer가 일시적으로 중단되더라도 데이터를 다시 읽을 수 있어 내구성과 내결함성이 보장된다.

**기존 메시징 시스템 및 데이터베이스와의 비교**
기존 메시징 시스템은 메시지를 소비하면 바로 제거되는 경우가 많았지만, Kafka는 메시지를 보유 기간 동안 유지한다. 따라서 Consumer가 오프라인 상태여도 데이터 유실 없이 복구가 가능하다.

또한 Kafka는 메시지 생산과 소비를 완전히 분리한다. Producer와 Consumer는 서로의 처리 속도에 영향을 주지 않고 독립적으로 동작할 수 있으며, 이는 시스템 안정성을 크게 향상시킨다.

**(번외) Eventual Consistency란**
분산 시스템에서 하나의 레코드를 여러 서버에 복제해 저장하는 경우, 데이터 변경 사항이 모든 서버에 즉시 반영되기는 어렵다. 데이터를 쓸 때 복제가 완료될 때까지 기다리는 방식은 **Strong Consistency를 제공**하지만, 지연시간이 증가한다.

반면 Kafka와 같은 시스템은 데이터를 쓰자마자 응답을 반환하고, 복제는 비동기적으로 진행된다. 이 경우 일부 서버에서는 아직 최신 데이터가 보이지 않을 수 있으며, 시간이 지나면 결국 일관된 상태에 도달한다. 이를 **Eventual Consistency**라고 한다.

![Eventual Consistency](/assets/img/contents/eventual_consistency.png "Eventual Consistency")

### **Kafka 주요 기능과 이점**
---
`kafka`는 처음부터 **스트림 처리**를 목적으로 설계된 플랫폼이다. `ksqlDB`를 활용하면 SQL 형태로 이벤트 데이터를 처리할 수 있다.

또한 kafka는 초당 수백만 건의 데이터를 처리할 수 있을 정도로 높은 처리량을 제공한다. 데이터 복제와 분산 커밋 로그 구조를 통해 **내결함성(Fault Tolerance)**을 확보하며, 브로커 추가만으로 손쉽게 확장 가능한 **확장성(Scalability)**을 갖추고 있다.

마지막으로 Kafka는 풍부한 생태계를 가진다. `Kafka Connect`를 통해 다양한 데이터 시스템과 연동할 수 있고, Schema Registry를 통해 이벤트 데이터의 스키마 관리도 가능하다. 이러한 점들이 Kafka를 실시간 데이터 처리의 표준 플랫폼 중 하나로 만들고 있다.

## Kafka 아키텍처
---
Kafka를 이해하는 핵심은 **데이터를 어떻게 스트림으로 관리하고, 이를 어떻게 확장성과 안정성을 갖춘 구조로 처리하는지**를 파악하는 데 있다. Kafka는 이벤트 데이터를 단순히 전달하는 것이 아니라, **구조화된 스트림 형태로 저장하고 소비할 수 있도록 설계**되어 있는지 알아본다.

### 데이터 이벤트 스트림
---
Kafka에서 데이터 이벤트 스트림은 `Topic`이라는 단위로 관리된다. Topic은 **이벤트가 지속적으로 쌓이는 논리적인 스트림**이며, `Producer`는 **특정 Topic에 이벤트를 기록**하고 `Consumer`는 **해당 Topic으로부터 데이터를 읽어들인다.**

**하나의 Topic은 다수의 Consumer가 동시에 읽을 수 있다.** 이를 통해 동일한 이벤트 스트림을 분석, 모니터링, 추천 시스템 등 여러 목적의 시스템에서 동시에 활용할 수 있다. 이 구조는 Kafka가 내부 데이터 버스로 사용될 수 있는 중요한 이유 중 하나다.

### Message (Event) 구조: Key, Value, Timestamp
---
Kafka에서 처리되는 메시지, 즉 이벤트는 기본적으로 `Key`, `Value`, `Timestamp`로 구성된다. 메시지의 크기는 최대 1MB까지 허용되며, Timestamp는 보통 해당 이벤트가 Topic에 기록된 시점을 의미한다.

`Key`는 단순한 문자열이 아니라, 복잡한 구조를 가질 수도 있다. 이 Key는 이후 메시지가 어느 Partition에 저장될지를 결정하는 데 사용되며, 데이터 파티셔닝 전략의 핵심 요소가 된다.

또한 Header는 선택적인 구성 요소로, 경량의 메타데이터를 key-value 형태로 저장할 수 있다. 이는 메시지의 본문과 분리된 추가 정보를 전달할 때 유용하다.

![Message Structure](/assets/img/contents/message_structure.png "Message Structure")

### Kafka 아키텍처 - Topic과 Partition
---
Kafka는 **확장성을 확보하기 위해 하나의 Topic을 여러 개의 Partition으로 나누어 저장**한다. `Partition`은 Kafka의 병렬 처리 단위이며, Partition의 수가 많을수록 더 높은 처리량을 기대할 수 있다.

메시지가 어떤 Partition에 저장될지는 Key의 유무에 따라 달라진다. Key가 존재하는 경우, Key의 해시 값을 Partition 수로 나눈 결과를 기준으로 Partition이 결정된다. 

이를 통해 동일한 Key를 가진 메시지는 항상 같은 Partition에 저장되며, 메시지 순서가 보장된다. 반면 Key가 없는 경우에는 라운드 로빈 방식으로 Partition이 선택되는데, 이는 순서 보장이 어렵기 때문에 일반적으로 권장되지 않는다.

![Topic Partition](/assets/img/contents/topic_partition.png "Topic Partition")

### Kafka 아키텍처 - Topic과 Partition과 복제본
---
각 Partition은 장애 대응을 위해 **복제본(Replica)**을 가진다. 이 구조를 통해 하나의 브로커에 장애가 발생하더라도 데이터 유실 없이 서비스를 유지할 수 있다.

Partition마다 하나의 `Leader`와 하나 이상의 `Follower`가 존재한다. **쓰기 작업**은 `Leader`를 통해서만 수행되며, **읽기 작업**은 설정에 따라 `Leader` 또는 `Follower`에서 수행될 수 있다.

Kafka는 Partition 단위로 일관성 수준을 설정할 수 있으며, 이는 **in-sync replica(ACK 설정)**를 통해 제어된다. 이를 통해 처리 지연과 데이터 안정성 사이의 균형을 시스템 요구사항에 맞게 조정할 수 있다.

![Topic Example](/assets/img/contents/topic_example.png "Topic Example")

### Kafka 아키텍처 - Broker: 실제 데이터를 저장하는 서버
---
Kafka 클러스터는 기본적으로 여러 대의 Broker로 구성된다. 이 Broker들은 **실제로 Topic의 Partition 데이터를 저장하고, Producer와 Consumer의 요청을 처리**한다. 하나의 Kafka 클러스터는 이론적으로 최대 약 20만 개의 Partition을 관리할 수 있으며, 이는 Broker들이 분산되어 Partition을 나눠 관리하기 때문에 가능한 구조다.

각 Broker는 최대 약 4,000개의 Partition을 처리할 수 있다. Broker는 물리 서버 또는 가상 머신 위에서 동작하며, Partition 데이터는 해당 서버의 디스크에 직접 기록된다. 따라서 Broker 수를 늘리면 자연스럽게 저장 용량과 처리량이 함께 증가하는 **Scale-Out 구조**를 갖는다.

이러한 수치적 제약은 Zookeeper를 사용하는 전통적인 Kafka 구성에서의 한계이며, 이를 개선하기 위해 Zookeeper를 대체하는 새로운 방식도 등장했다.

### Kafka 아키텍처 - Broker와 Partition
---
Kafka Broker는 Kafka Server 혹은 Kafka Node라고도 불린다. Broker의 핵심 역할은 Topic을 구성하는 Partition들을 실제로 관리하는 것이다. 각 Partition은 특정 Broker에 할당되어 저장되며, 해당 Broker가 Partition의 Leader 또는 Follower 역할을 수행한다.

이처럼 Partition과 Broker의 매핑 정보는 Kafka 클러스터 전체에서 공유되어야 하며, 이를 위해 메타데이터 관리가 필수적이다.

![broker_partition](/assets/img/contents/broker_partition.png "broker_partition")

### Kafka 아키텍처 - 메타 정보 관리를 어떻게 할 것인가?
---
Kafka는 단순히 메시지를 저장하는 시스템이 아니라, 다양한 메타정보를 함께 관리한다. 예를 들어 **어떤 Broker들이 클러스터에 참여하고 있는지에 대한 Broker 리스트(Broker Membership)**, 그리고 그중 **어떤 Broker가 클러스터의 상태를 관리하는 Controller**인지에 대한 정보가 필요하다.

또한 Topic 리스트와 Topic 설정 정보, 각 Topic을 구성하는 Partition과 그 Replica의 상태 역시 관리 대상이다. 이때 Partition과 Replica 관리는 Controller가 담당하며, Controller는 Broker 중 하나가 맡는다.

이 외에도 **Topic별 접근 권한을 제어하기 위한 ACL(Access Control Lists)**, 그리고 **클라이언트의 과도한 사용을 제한하기 위한 Quota 관리** 역시 중요한 메타데이터다.

### Kafka 아키텍처: Zookeeper와 Controller
Kafka 0.8.2 버전(2015년)부터는 `Controller` 개념이 도입되었다. Controller는 **Broker이면서 동시에 Partition 관리와 리더 선출을 담당하는 역할을 수행**한다. Kafka의 장기적인 목표는 Zookeeper 의존도를 줄이거나 완전히 제거하는 것이며, 현재는 두 가지 운영 모드가 공존한다.

첫 번째는 `Zookeeper` 모드이다. 이 방식에서는 3대, 5대, 혹은 7대의 서버로 Zookeeper Ensemble을 구성한다. Zookeeper는 메타데이터 저장과 Controller 선출을 담당하며, 클러스터 내에는 항상 하나의 Controller만 존재한다.

두 번째는 `KRaft` 모드이다. KRaft 모드에서는 Zookeeper를 완전히 배제하고, Kafka 자체가 메타데이터 관리와 컨트롤 기능을 수행한다. 이 방식에서는 다수의 Controller가 존재하며, 이들이 Zookeeper의 역할을 대체한다. 일반적으로 Controller들은 Broker 역할도 함께 수행한다.

### (번외) Zookeeper란
---
Zookeeper는 분산 시스템에서 널리 사용되어 온 **Distributed Coordination Service**다. 여러 노드로 구성된 시스템에서 동기화, 설정 관리, 리더 선출과 같은 작업을 중앙에서 조율하기 위한 목적으로 설계되었다. 분산 환경에서 각 노드가 동일한 상태를 인식하고 협력할 수 있도록 돕는 역할을 한다.

Zookeeper는 원래 Yahoo!의 Hadoop 프로젝트 일부로 자바 기반으로 개발되었으며, 이후 Apache 오픈소스 프로젝트로 발전했다. 이로 인해 **오랫동안 다양한 분산 시스템의 핵심 구성 요소**로 사용되어 왔다.


**Zookeeper의 한계와 문제점**
Zookeeper는 분산 시스템의 조율을 담당하는 데 효과적이지만, 구조적인 한계도 분명하다. 먼저 **Zookeeper는 지원하는 데이터 크기가 매우 작고, 동기 방식으로 동작**한다. 이로 인해 **처리 속도가 느리고, 일정 규모 이상으로 확장할 경우 병목이 발생하기 쉽다.**

또한 설정과 운영이 복잡하다는 점도 문제로 지적된다. 안정적인 운영을 위해 여러 대의 서버로 Ensemble을 구성해야 하며, 장애 대응과 설정 관리에 상당한 부담이 따른다. 이러한 이유로 점차 많은 서비스들이 Zookeeper 의존도를 줄이거나, 아예 다른 방식으로 대체하기 시작했다.

Kafka 역시 이러한 흐름 속에서 Zookeeper를 제거하려는 방향으로 발전하고 있으며, ElasticSearch 또한 Zookeeper를 사용하다가 자체적인 메타데이터 관리 방식으로 전환한 대표적인 사례다.

**Zookeeper의 주요 사용 사례**
그럼에도 불구하고 Zookeeper는 오랫동안 다양한 분산 시스템에서 핵심 역할을 수행해왔다. 대표적으로 메시지 큐 시스템인 Apache Kafka, 분산 데이터베이스 조정을 위한 `Apache HBase`, 분산 스트림 처리 시스템인 `Apache Storm` 등에서 사용되었다.

이들 시스템에서 Zookeeper는 노드 간 상태 공유, 리더 선출, 설정 정보 관리와 같은 기능을 담당하며 분산 환경의 안정성을 유지하는 데 기여했다.

## Kafka 주요 개념
---
Kafka는 메시지를 단순히 전달하는 시스템이 아니라, **로그 기반 스토리지 모델을 중심으로 설계된 스트리밍 플랫폼**이다. 이를 이해하기 위해서는 Topic, Partition, Segment로 이어지는 계층 구조와 데이터 저장 방식의 특성을 함께 살펴볼 필요가 있다.

![Kafka Architecture 1](/assets/img/contents/kafka_architecture_1.png "Kafka Architecture 1")

![Kafka Architecture 2](/assets/img/contents/kafka_architecture_2.png "Kafka Architecture 2")

### Topic의 기본 동작 방식
---
Kafka에서 Topic은 Consumer가 데이터를 읽는 단위지만, **Consumer가 데이터를 읽는다고 해서 메시지가 사라지지는 않는다.** Topic에 저장된 데이터는 설정된 보유 기간(retention period) 동안 유지되며, 여러 Consumer가 동일한 데이터를 각자의 속도로 읽을 수 있다.

Kafka는 Consumer별로 **어디까지 데이터를 읽었는지에 대한 위치 정보(offset)**를 유지한다. 이 정보는 장애 상황에서도 복구가 가능하도록 중복 저장되며, Kafka의 내결함성을 구성하는 중요한 요소다.

### Topic, Partition, Replication 구조
---
하나의 Topic은 확장성을 위해 여러 개의 `Partition`으로 나뉜다. Partition은 **Kafka의 병렬 처리 단위**이며, Partition 수를 늘리면 처리량을 수평적으로 확장할 수 있다.

각 Partition은 장애 대응을 위해 **복제본(Replication Partition)**을 가진다. Partition마다 하나의 **Leader와 하나 이상의 Follower가 존재하며, 쓰기 작업은 Leader를 통해서만 이루어진다.** 읽기 작업은 설정에 따라 Leader 또는 Follower를 통해 수행될 수 있다. 이 구조를 통해 Kafka는 장애 발생 시에도 빠르게 Fail-over가 가능하다.

### Partition과 Segment의 관계
---
하나의 Partition은 다시 여러 개의 `Segment`로 구성된다. **Segment는 변경되지 않고 데이터가 계속 추가되는 Append-Only 로그 파일**로, Kafka의 커밋 로그(Commit Log)를 구성하는 기본 단위다.

각 Segment는 디스크 상의 하나의 파일이며, 크기에 제한이 있다. Segment 파일이 최대 크기에 도달하면 새로운 Segment가 생성된다. 이로 인해 각 Segment는 자신만의 데이터 오프셋 범위를 가지게 된다.

### 로그 파일의 특성
---
Kafka의 로그 파일, 정확히는 Segment의 특성은 매우 단순하면서도 강력하다. 데이터는 항상 파일의 끝에만 추가되며(Append Only), 한 번 기록된 데이터는 변경되지 않는다(Immutable).

데이터는 설정된 Retention Period에 따라 일정 시간이 지나면 삭제되며, 각 메시지에는 순서를 나타내는 Offset 번호가 부여된다. 이 Offset을 기준으로 Consumer는 자신이 어디까지 데이터를 처리했는지를 정확히 추적할 수 있다.

### Broker의 역할
---
Kafka에서 Topic은 시간 순서대로 정렬된 메시지들의 집합이다. `Producer`는 먼저 **Topic을 생성하고 필요한 속성을 지정한 뒤, 메시지를 `Broker`로 전송**한다. `Broker`는 **수신한 메시지를 Topic에 속한 Partition으로 나누어 저장**하며, 이때 장애 대응을 위해 **Replication Factor**에 따라 **Leader와 Follower 복제본을 함께 관리**한다.

Consumer는 **직접 Producer와 통신하지 않고, 항상 Broker를 통해 메시지를 읽는다.** 이 구조를 통해 생산과 소비가 분리되며, 시스템 전체의 안정성이 확보된다.

Kafka 클러스터는 **여러 개의 Broker로 구성**된다. 하나의 Broker는 여러 Partition을 동시에 관리하며, 각 Partition 데이터는 Broker가 실행 중인 서버의 디스크에 저장된다. **Topic에 속한 메시지들은 스케일 아웃을 위해 여러 Partition에 분산 저장되고, 이 Partition과 Replica들의 전체적인 배치는 Controller가 관리**한다.

개념적으로 하나의 Partition은 하나의 로그 파일로 볼 수 있으며, 각 메시지는 고유한 위치 정보인 Offset을 가진다. 메시지의 저장 기간은 Retention Policy에 의해 제어된다.

### Producer와 Partition 선택
---
하나의 Topic은 여러 Partition으로 구성되며, 어떤 Partition에 메시지를 저장할지는 Producer가 결정한다. Partition은 두 가지 목적을 가진다. 첫째는 로드 밸런싱을 통한 처리량 확장이고, 둘째는 특정 Key를 기준으로 메시지를 묶는 **의미적 파티셔닝(Semantic Partitioning)**이다.

Producer는 기본적으로 `hash(key) % partition 수` 방식으로 Partition을 선택한다. Key가 없는 경우에는 라운드 로빈 방식이 사용되며, 필요에 따라 커스텀 Partition 로직을 구현할 수도 있다. Partition 전략은 메시지 순서 보장과 처리 성능에 직접적인 영향을 미친다.

### Consumer의 기본 개념
---
`Consumer`는 **Topic을 구독(Subscription)하여 메시지를 읽는다.** Consumer는 자신이 마지막으로 읽은 메시지의 Offset을 관리하며, 이를 통해 중단 이후에도 정확한 위치에서 다시 처리할 수 있다. Kafka는 이를 지원하기 위한 커맨드라인 Consumer 유틸리티도 제공한다.

Consumer는 **Consumer Group이라는 개념을 통해 수평 확장이 가능**하다. 하나의 Consumer Group 내에서 Partition은 Consumer들에게 나뉘어 할당되며, 이를 통해 처리량을 늘리고 Backpressure 문제를 완화할 수 있다.

실무에서는 하나의 프로세스가 Consumer이면서 동시에 Producer 역할을 수행하는 경우도 매우 흔하다. 이 방식으로 **데이터를 가공한 뒤 새로운 Topic으로 다시 발행하는 구조는 Kafka 기반 데이터 파이프라인의 대표적인 패턴**이다.

## Kafka 기타 기능
---
Kafka가 실시간 데이터 파이프라인의 중심 역할을 하게 되면서, Kafka 자체뿐 아니라 Kafka를 둘러싼 주변 생태계 구성 요소의 중요성도 커졌다. 그중 대표적인 것이 `Kafka Connect`와 `Kafka Schema Registry`다. 이 두 구성 요소는 Kafka를 기존 데이터 시스템과 안전하게 연결하는 역할을 담당한다.

### Kafka Connect
---
Kafka Connect는 **Kafka 위에 구축된 중앙 집중형 데이터 통합 허브**이다. Kafka를 단순한 메시징 시스템이 아닌, 데이터 시스템 간의 **데이터 버스(Data Bus)**로 활용할 수 있도록 돕는 별도의 오픈소스 프로젝트이다. Kafka Connect는 Kafka Broker와는 별도로 동작하며, 이를 위해 전용 서버들이 필요하다.

Kafka Connect는 두 가지 실행 모드를 제공하는데, `Standalone` 모드는 주로 개발과 테스트 용도로 사용되며, `Distributed` 모드는 실제 운영 환경에서 사용된다. 운**영 환경에서는 장애 대응과 확장성을 위해 Distributed 모드가 일반적**이다.

Kafka Connect의 핵심 목적은 다양한 데이터 시스템 간의 데이터를 주고받는 것이다. 데이터베이스, 파일 시스템, 키-값 저장소, 검색 인덱스 등과 Kafka를 연결하여, 외부 데이터를 Kafka로 가져오거나 Kafka의 데이터를 외부 시스템으로 지속적으로 전달할 수 있다. 이때 **데이터를 가져오는 쪽**을 `Source`, **내보내는 쪽**을 `Sink`라고 부른다.

**Kafka Connect의 내부 구조**
Kafka Connect는 Broker 중 일부 서버나, 완전히 별도의 서버들로 구성될 수 있다. Kafka Connect 클러스터 내부에서는 `Worker`들이 동작하며, **실제 데이터 이동 작업**은 Worker가 수행하는 `Task` 단위로 나뉜다.

`Task`는 역할에 따라 **Source Task와 Sink Task**로 구분된다. Source Task는 **외부 데이터 소스로부터 데이터를 읽어 Kafka의 이벤트 스트림으로 변환**하고, Sink Task는 **Kafka에 저장된 데이터를 외부 시스템으로 전달**한다. 예를 들어 Kafka의 데이터를 S3 버킷에 지속적으로 저장하는 작업은 Kafka Connect를 통해 매우 쉽게 구성할 수 있다.

![Kafka Connect](/assets/img/contents/kafka_connect.png "Kafka Connect")

### Kafka Schma Registry
---
Kafka 기반 **시스템에서 이벤트 데이터가 많아질수록, 메시지 구조의 일관성과 변경 관리가 중요**한 문제가 된다. Kafka Schema Registry는 Topic에 저장되는 메시지 데이터의 스키마를 중앙에서 관리하고 검증하기 위한 서비스다.

Producer와 Consumer는 **Schema Registry를 통해 스키마를 공유**하며, 스키마 변경이 발생하더라도 안전하게 처리할 수 있다. 이를 통해 **데이터 포맷 불일치로 인한 장애를 사전에 방지**할 수 있다.

Schema Registry는 Schema ID와 버전을 기반으로 **스키마 진화(Schema Evolution)**를 지원한다. 일반적으로 AVRO 포맷이 많이 사용되며, Protobuf나 JSON도 함께 사용된다.

스키마 변경 시에는 호환성 전략을 선택해야 한다. Producer를 먼저 변경하고 Consumer를 점진적으로 수정하는 방식은 Forward Compatibility, Consumer를 먼저 변경하는 방식은 **Backward Compatibility**라고 한다. 양쪽 모두를 동시에 고려하는 방식은 Full Compatibility다.

**(번외) Serialization and Deserialization (직렬화 & 역직렬화)**
메시지를 Kafka로 전송하기 위해서는 데이터를 **직렬화(Serialization)**해야 한다. 직렬화는 **객체의 상태를 저장하거나 전송 가능한 형태로 변환하는 과정**으로, 이때 **데이터 압축이 함께 이루어지기도 하며, 스키마 정보가 포함**될 수 있다.

Consumer 측에서는 **역직렬화(Deserialization)** 과정을 통해 **직렬화된 데이터를 다시 사용할 수 있는 형태로 변환**한다. 이 과정에서 압축 해제와 함께 스키마 검증이 수행된다. 이러한 직렬화와 역직렬화 작업은 보통 Kafka 관련 라이브러리들이 담당한다.

### Kafka 아키텍처 - REST Proxy
---
`Kafka REST Proxy`는 **HTTP API를 통해 Kafka를 사용할 수 있도록 해주는 컴포넌트**다. 이를 통해 클라이언트는 Kafka 전용 라이브러리를 사용하지 않고도 메시지를 생성하거나 소비하고, 토픽을 관리할 수 있다. 표준화된 REST API를 제공하기 때문에 언어와 환경에 대한 제약이 크게 줄어든다.

`REST Proxy`는 **메시지의 직렬화와 역직렬화를 대신 수행하며, 내부적으로 로드 밸런싱** 역할도 담당한다. 특히 사내 네트워크 외부에서 Kafka에 접근해야 하는 경우에 유용하며, Kafka 클러스터를 직접 노출하지 않고도 안전한 접근 지점을 제공할 수 있다.

### Kafka 아키텍처 - Streams와 KSQL
---
Kafka Streams는 Kafka Topic을 소비하고 다시 Kafka Topic으로 **결과를 생성하는 실시간 스트림 처리 라이브러리**다. 별도의 클러스터를 구성할 필요 없이 애플리케이션 라이브러리 형태로 사용 가능하다는 점이 특징이다.

`Spark Streaming`으로 Kafka 데이터를 처리하는 경우가 **마이크로 배치 기반**에 가깝다면, `Kafka Streams`는 **레코드 단위 처리를 중심으로 하여 보다 실시간에 가까운 처리를 제공**한다. 이로 인해 지연시간이 짧고, 비교적 단순한 스트림 처리 로직에 적합하다.

### Kafka 아키텍처 - ksqlDB
---
KSQL은 Confluent에서 개발한 Kafka용 오픈소스 SQL 엔진으로, **스트리밍 데이터를 SQL로 처리할 수 있도록 설계**되었다. 이를 통해 **사용자는 연속적으로 유입되는 데이터를 대상으로 Continuous Query를 작성하고, 실시간 분석과 변환을 수행**할 수 있다.

이후 KSQL은 `ksqlDB`로 발전했다. `ksqlDB`는 **Kafka Streams를 기반으로 구현된 스트림 처리 데이터베이스**로, SQL과 유사한 쿼리 언어를 제공한다. 필터링, 집계, 조인, 윈도우 연산 등 일반적인 SQL 작업을 실시간 스트림 데이터에 적용할 수 있다.

ksqlDB의 중요한 특징은 연속 쿼리와 지속적으로 업데이트되는 뷰를 지원한다는 점이다. **데이터가 도착하는 즉시 결과가 갱신되며, 이를 통해 실시간 집계와 변환을 손쉽게 구현**할 수 있다. 이는 Spark 환경에서 SQL 기반 분석이 대세가 된 흐름과도 유사하다.

## Kafka 프로그래밍 with Python
---
Kafka를 사용하기 위한 각 프로그래밍 언어들의 옵션은 다음과 같다:

- `Java`:
    - Apache Kafka Java Client: 아파치 카프카의 공식 Java 클라이언트 라이브러리
    - Spring Kafka: 스프링 프레임워크와 Kafka를 통합하기 위한 라이브러리

- `Python`:
    - Confluent Kafka Python: Confluent에서 개발한 공식 Kafka Python 클라이언트 라이브러리
    - Kafka-Python: 또다른 파이썬 기반 라이브러리

- `.NET`:
    - Confluent Kafka .NET Client: Confluent에서 개발한 공식 Kafka .NET 클라이언트 라이브러리

- `GO`:
    - Sarama: Go 언어용 Kafka 클라이언트 라이브러리

- `Node.js`:
    - node-rdkafka: librdkafka를 기반으로 한 Node.js용 Kafka 클라이언트 라이브러리
    - kafka-node: Node.js용 Kafka 클라이언트 라이브러리

### Kafka Python 프로그래밍 기본
---

**Kafka 설치 - Docker Compose 사용**

```shell
$ git clone https://github.com/conduktor/kafka-stack-docker-compose.git

$ cd kafka-stack-docker-compose

$ docker-compose -f full-stack.yml up
```
Docker Compose yaml 파일은 다음과 같은 구조로 이루어진디:

```yaml
version: '2.1'
services:
    zoo1:
    kafka1:
    kafka-schema-registry:
    kafka-connect:
    ksqldb-server:
    conduktor-platform:
volumes:
    ...
```

**Python 모듈 설치**

```shell
pip3 install kafka-python
```

**간단한 Producer 만들기**

```python
from time import sleep
from json import dumps
from kafka import KafkaProducer

# 로컬 Kafka 인스턴스를 연결하는 KafkaProducer 객체 생성
# 전송하려는 데이터를 json 문자열로 변환한 뒤, 
# UTF-8로 인코딩하여 직렬화 방법 정의
producer = KafkaProducer(
    # Broker들 중 하나 이상을 지정
    bootstrap_servers = ['localhost:9092'],
    value_serializer = lambda x: dumps(x).encode('utf-8')
)

# 0.5초마다 "topic_test"라는 토픽과 반복 카운터를 데이터로 포함하는 이벤트를 전송
# 키-값 데이터로 'counter'라는 키와 정수를 값으로 갖도록 구성함
for j in range(999):
    print("Iteration", j)
    data = {'counter': j}
    # key와 headers는 지정되어 있지 않음
    producer.send('topic_test', value=data)
    sleep(0.5)
```

**Consumer 객체 만들기**

```python
from time import sleep
from json import loads
from kafka import KafkaConsumer

# 로컬 Kafka 인스턴스를 연결하는 KafkaConsumer 객체 생성
# "topic_test" 트픽에서 가장 먼저 생긴 데이터를 읽고,
# 오프셋 정보는 계속해서 업데이트 후, 
#  my-group-id라는 이름의 consumer group에 조인하도록 설정
consumer = KafkaConsumer(
    'topic_test',
    bootstrap_servers = ['localhost:9092'],
    auto_offset='earliest'
    # 만약 False일 경우, commit 함수를 명시적으로 offset 위치를 commit해야 함
    enable_auto_commit=True,
    group_id='my_group_id',
    # Producer에서 수행한 value_serializer의 반대 작업 수행
    value-deserializer=lambda x: loads(x.decode('utf-8'))
)

# 2초마다 "topic-test"라는 토픽에서 카운터 값을 읽도록 구성
for event in consumer:
    event_data = event.value
    print(event_data)
    sleep(2)
```
### Kafka CLI Tools 접근
---
`docker ps`를 통해 Broker의 Container ID 혹은 Container 이름을 파악하여, 해당 컨테이너로 로그인한다.

```shell
docker exec -it Broker_Container_ID sh
```

해당 shell에서 다양한 Kafka 관련 클라이언트 툴을 사용할 수 있다.

```shell
$ kafka-topics --bootstrap-server kafka1:9092 --list
$ kafka-topics --bootstrap-server kafka1:9092 --delete --topic topic_test
```

Command line을 통해 `Topic`을 만들고, `Message` 생성이 가능하다.

```shell
$ kafka-console-producer --bootstrap-server kafka1:9092 --topic test_console
```

Command line을 통해 `Topic`에서 `Message` 읽기가 가능하다. 만약 `--from-beginning` 옵션이 있다면, 처음부터 읽음 (earliest). 아니면 latest로 동작하게 된다.

```shell
kafka-console-consumer --bootstrap-server kafka1:9092 --topic test_console --from-beginning
```

### Topic 파라미터 설정
---
Topic 생성시 다수의 Partition이나 Replica를 주려면, 먼저 `KafkaAdminClient` 오브젝트를 생성하고 create_topics 함수로 `Topic`을 추가하고, create_topics의 인자로는 `NewTopic` 클래스의 오브젝트를 지정한다.

```python
clinet = KafkaAdminClient(bootstrap_servers=bootstrap_servers)
topic = NewTopic(
    name=name,
    num_partitions=partitions,
    replication_factor=replica
)

client.create_topics([topic])
```


### Kafka Producer 동작 파라미터
---
Kafka Producer의 동작을 위한 파라미터들은 다음과 같이 구성된다:

| 파라미터 | 의미 | 기본 값 |
|---|---|---|
| bootstrap_servers | 메시지를 보낼 때 사용할 브로커 리스트 (host:port) | localhost:9092 |
| client_id | Kafka Producer의 이름 | kafka-python-{version} |
| key_serializer, value_serializer | 메시지의 키와 값을 직렬화(serialize)하는 방법을 지정하는 함수 | |
| enable_idempotence | 중복 메시지 전송을 막을 것인지 여부 | False |
| acks (0, 1, all) | Consistency level (0: 바로 리턴, 1: Leader에 기록 시 리턴, all: 모든 Replica에 기록될 때까지 대기) | 0 |
| retries, delivery.timeout.ms | 메시지 실패 시 재시도 횟수 / 메시지 전송 최대 시간(ms) | 2147483647, 120000 |
| linger_ms, batch_size | 배치 전송 설정 (송신 전 대기 시간(ms), 송신 전 데이터 크기(bytes)) | 0, 16384 |
| max_in_flight_requests_per_connection | Broker 응답을 기다리지 않고 전송 가능한 최대 메시지 수 | 5 |

각 파라미터들은 다음과 같은 흐름으로 동작하게 된다:

![Kafka Producer 동작](/assets/img/contents/kafka_producer.png "Kafka Producer 동작")

### Kafka Consumer 동작 파라미터
---
Kafka Consumer의 동작을 위한 파라미터들은 다음과 같이 구성된다:

| 파라미터 | 의미 | 기본 값 |
|---|---|---|
| bootstrap_servers | 메시지를 받을 때 사용할 브로커 리스트 (host:port) | localhost:9092 |
| client_id | Kafka Consumer의 이름 | kafka-python-{version} |
| group_id | Kafka Consumer Group의 이름 | |
| key_deserializer, value_deserializer | 메시지의 키와 값을 역직렬화(deserialize)하는 방법을 지정하는 함수 | |
| auto_offset_reset | 초기 오프셋이 없을 때 읽기 시작 위치 (earliest 또는 latest) | latest |
| enable_auto_commit | True이면 소비자의 오프셋을 백그라운드에서 주기적으로 커밋, False이면 명시적으로 커밋 필요 (오프셋은 별도 리셋 가능) | True |

### Consumer는 여러 Partition을 어떻게 읽는가
---
하나의 Consumer가 있고, 해당 Consumer가 다수의 Partition으로 구성된 Topic을 읽어야 하는 상황을 가정해보자. 이 경우 Consumer는 각 Partition으로부터 라운드 로빈 방식으로 하나씩 메시지를 읽게 된다.

이 구조에서는 병렬성이 떨어질 수 있다. 데이터 생산 속도가 빠를수록 Consumer가 이를 따라가지 못하게 되고, 결과적으로 Backpressure가 심해질 가능성이 커진다. 이러한 문제를 해결하기 위해 등장한 개념이 바로 **Consumer Group**이다.

한편, 하나의 프로세스에서 여러 Topic을 읽는 것도 가능하다. 이 경우 Topic 수만큼 KafkaConsumer 인스턴스를 생성해야 하며, 각각 다른 Group ID와 Client ID를 지정하는 것이 일반적이다.

**Consumer Group이란 무엇인가**
Consumer Group은 Kafka에서 **데이터 소비의 병렬성과 장애 대응을 동시에 해결하기 위한 핵심 개념**이다. Consumer가 Topic을 읽기 시작하면, 해당 Topic의 Partition들이 Consumer Group 내의 Consumer들에게 자동으로 할당된다.

Partition 수가 Consumer 수보다 많다면, Partition들은 라운드 로빈 방식으로 Consumer들에게 분배된다. 이때 중요한 제약은 **하나의 Partition은 하나의 Consumer에게만 할당된다**는 점이다. 이를 통해 데이터 중복 처리 없이 병렬 소비가 가능해진다.

이 구조의 장점은 명확하다. 데이터 소비 병렬성이 증가하여 Backpressure를 완화할 수 있고, 일부 Consumer가 중단되더라도 나머지 Consumer들이 계속해서 데이터를 처리할 수 있다.

**Consumer Group Rebalancing**
Consumer Group 내에서 Consumer의 수가 변하면, Partition 할당을 다시 조정해야 한다. 기존 Consumer가 장애로 사라지거나, 새로운 Consumer가 Group에 참여하는 경우가 이에 해당한다.

이러한 재할당 과정을 **Consumer Group Rebalancing**이라고 하며, Kafka가 이를 자동으로 수행한다. Rebalancing은 편리하지만, 그 동안 일시적인 처리 중단이 발생할 수 있기 때문에 Consumer 설계 시 이를 고려해야 한다.

### 메시지 처리 보장 방식 (Delivery Semantics)
---
실시간 메시지 처리 시스템에서는 메시지가 얼마나 정확하게 전달되고 처리되는지를 정의하는 보장 방식이 중요하다. 일반적으로 세 가지 방식이 존재한다.

**At Most Once**
At Most Once는 메시지가 **최대 한 번만 처리됨을 보장**한다. 중복은 없지만, 메시지 손실 가능성이 존재한다. 가장 단순한 방식이며, 일부 로그 처리나 중요도가 낮은 데이터에 사용된다.

**At Least Once**
At Least Once는 모든 메시지가 적어도 한 번 이상 처리됨을 보장한다. 대신 메시지 중복 가능성이 있으며, Consumer는 중복 처리를 방지하기 위한 **멱등성(Idempotency) 로직을 직접 구현**해야 한다. 보통 Consumer가 오프셋을 직접 커밋하는 경우 이 방식이 사용된다.

**Exactly Once**
Exactly Once는 **각 메시지가 정확히 한 번만 처리됨을 보장**한다. 가장 이상적인 방식이지만, 네트워크 장애나 재시도 상황까지 고려해야 하므로 구현 난이도가 매우 높다. Kafka에서는 Producer 측에서 `enable.idempotence`를 활성화하고, Producer와 Consumer 모두 Transaction API를 사용하는 방식으로 이를 지원한다.

![Message Processing Guarantee](/assets/img/contents/message_processing.png "Message Processing Guarantee")

### ksqlDB 사용 예제
---
RESTAPI나 ksql 클라이언트 툴을 사용하여 Topic을 테이블처럼 SQL(ksql)로 조작하는 간단한 예제를 진행한다.

```shell
# confluentinc/cp-ksqldb-server의 Container ID 복사를 위해 실행 중인 docker container list 출력
$ docker ps

$ docker exec -it ContainerID sh
```

ksql 실행 후 두 개의 명령어를 실행한다:

```sql
CREATE STREAM my_stream (id STRING, name STRING, title STRING) with (kafka_topic='fake_people', value_format='JSON');

SELECT * FROM my_stream;
```

## Spark Streaming 소개
---
Spark Streaming은 **실시간 데이터 스트림 처리를 위한 Spark의 API**다. Kafka, Kinesis, Flume, TCP 소켓 등 다양한 소스로부터 유입되는 데이터를 처리할 수 있으며, 배치 처리에서 사용하던 Join, Map, Reduce, Window와 같은 고급 연산을 그대로 사용할 수 있다.

이로 인해 기존 배치 처리 로직을 크게 변경하지 않고도 실시간 처리로 확장할 수 있다는 장점이 있다.

**Spark Streaming의 동작 방식**
Spark Streaming은 데이터를 완전히 실시간으로 처리하기보다는, **마이크로 배치(Micro-batch) 방식으로 처리**한다. 일정 시간 동안 수집된 데이터를 하나의 작은 배치로 묶어 처리하고, 이 과정을 반복적으로 수행한다.

각 배치마다 데이터의 시작과 끝 위치를 관리하며, 이전 배치에서 처리된 데이터와 병합해 전체 스트림 상태를 유지한다. 장애가 발생할 경우에는 데이터를 다시 처리할 수 있도록 설계되어 있어, Fault Tolerance와 데이터 재처리가 가능하다.

**Spark Streaming의 내부 동작**
내부적으로 Spark Streaming은 **실시간 입력 스트림을 여러 개의 배치로 나눈 뒤, 이를 Spark Engine에서 처리**한다. 처리 결과는 다시 스트림 형태로 이어져 최종 결과를 만들어낸다.

Spark Streaming에는 두 가지 주요 추상화가 존재한다. 초기 방식인 `DStream`과, 이후 등장한 `Structured Streaming`이다. 

|DStream|Structured Streaming|
|---|---|
|RDD 기반 스트리밍 처리|DataFrame 기반 스트리밍 처리|
|Spark SQL 엔진의 최적화 기능 사용 불가|Catalyst 기반 최적화 혜택을 가져감|
|이벤트 발생 시간 기반 처리 불가|이벤트 발생 시간 기반 처리 가능|
|개발이 중단된 상태|계속해서 기능이 추가되고 있음|

**Structured Streaming은 더 선언적인 API와 강력한 최적화 기능을 제공**하며, 현재 Spark 실시간 처리의 중심 모델로 자리 잡고 있다.

![Spark Streaming 내부 구조](/assets/img/contents/spark_streaming_internal.png "Spark Streaming 내부 구조")

### Source & Sink
---
Spark Structured Streaming에서 스트리밍 처리를 이해하는 핵심은 **Source와 Sink** 개념이다. Source와 Sink는 **외부 시스템과 Spark 사이에서 데이터를 주고받는 역할**을 하며, **스트리밍 파이프라인의 시작과 끝을 구성**한다.

![Spark Streaming 전체 구조](/assets/img/contents/spark_streaming_architecture.png "Spark Streaming 전체 구조")

**Source**
Source는 외부 시스템에서 발생하는 스트리밍 데이터를 Spark Structured Streaming으로 수집할 수 있도록 해주는 구성 요소다. Kafka, Amazon Kinesis, Apache Flume, TCP/IP 소켓, HDFS, 파일 시스템 등 다양한 데이터 소스를 지원한다.

Structured Streaming에서 Source를 통해 읽어온 데이터는 결국 **Spark DataFrame**으로 변환된다. 이 덕분에 스트리밍 데이터라 하더라도 배치 처리와 동일한 DataFrame API를 사용할 수 있다. 예를 들어 Kafka에 저장된 데이터를 Spark Structured Streaming으로 수집하려는 경우, Kafka Source를 사용해 하나 이상의 Topic에서 데이터를 읽어 DataFrame 형태로 변환할 수 있다.

배치 처리와의 가장 큰 차이점은 `read`가 아닌 `readStream` API를 사용한다는 점이다. 이를 통해 Spark는 해당 DataFrame이 지속적으로 갱신되는 스트리밍 데이터임을 인식하게 된다.

```python
lines_df = spark.readStream \
    .format("socket") \
    .option("host", "localhost") \
    .option("port", "9999") \
    .load()
```

**Sink**
Sink는 Spark Structured Streaming에서 처리된 데이터를 **외부 시스템이나 스토리지로 출력하는 역할**을 한다. 즉, 스트리밍 파이프라인의 결과가 어디로 전달되고, 어떻게 소비될지를 정의한다.

Sink 역시 Source와 마찬가지로 다양한 대상 시스템을 지원한다. Kafka, HDFS, Amazon S3, Apache Cassandra, JDBC 기반 데이터베이스 등이 대표적인 예다. 예를 들어 Kafka Sink를 사용하면 Spark Structured Streaming에서 처리된 데이터를 다시 Kafka Topic으로 전송할 수 있다.

```python
word_count_query = counts_df.writeStream \
.format("console") \
.outputMode("complete") \
.option("checkpointLocation", "chk-point-dir") \
.start()
```

**(추가) Output Mode**
Sink로 데이터를 출력할 때는 **Output Mode를 통해 현재 마이크로 배치의 결과가 어떻게 반영될지를 결정**한다.

- `Append` 모드: 새로운 데이터만 추가하는 방식으로, 변경되지 않는 결과에 적합하다.
- `Update` 모드: 기존 결과를 갱신하는 방식으로, UPSERT와 유사한 동작을 한다.
- `Complete` 모드: 전체 결과를 매번 다시 쓰는 방식으로, 일종의 FULL REFRESH에 해당한다.

Output Mode 선택은 처리 로직과 Sink의 특성에 따라 신중하게 결정해야 한다.