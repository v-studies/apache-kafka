
# 3장 카프카 기본 개념 설명

## 3.1 카프카 브로커, 클러스터, 주키퍼

### 데이터 복제

카프카의 데이터 복제는 파티션 단위로 이루어진다.

<img width="640" height="292" alt="Image" src="https://github.com/user-attachments/assets/9f1532ee-0451-40e7-a419-a25913741de6" />

- 팔로워는 주기적으로 리더의 오프셋을 확인한다.
- 자신의 오프셋이 뒤쳐져 있으면, 리더로부터 누락된 데이터를 가져와 저장함
- 이 동작을 복제(Replication)라고 부른다.

### 컨트롤러

클러스터의 다수 브로커 중 한 대가 컨트롤러의 역할을 한다. 

- 클러스터의 메타데이터를 관리하고, 브로커/파티션 리더 장애 시 새로운 리더를 선출하는 역할을 한다.
- 클러스터 전체에 단 하나만 존재

### 코디네이터

클러스터의 다수 브로커 중 한 대는 코디네이터의 역할을 수행한다.

- 코디네이터는 컨슈머 그룹의 상태를 체크하고 파티션을 컨슈머와 매칭되도록 분배하는 역할을 한다.
- 컨슈머 변화 발생 시, 매칭되지 않은 파티션을 정상 컨슈머에게 재할당
- 컨슈머 그룹 단위로 존재

<br>

## 3.2 토픽과 파티션

### 토픽

카프카에서 데이터를 구분하기 위한 논리적 단위.

토픽은 1개 이상의 파티션을 소유하고 있다.

### 파티션

파티션은 카프카의 병렬처리의 핵심으로써 그룹으로 묶인 컨슈머들이 레코드를 병렬로 처리할 수 있도록 매칭된다.

- 파티션은 자료구조에서 접하는 큐와 비슷한 구조라고 생각하면 쉽다.
- FIFO 구조와 같이 먼저 들어간 레코드는 컨슈머가 먼저 가져가게 된다.

<br>

## 3.3 레코드

- 레코드는 카프카의 데이터 단위 (타임스탬프, 키, 값, 오프셋, 헤더)
- key = 파티션 분배 기준 (같은 키는 같은 파티션 → 순서 보장)
- offset = 파티션 내 고유 번호, 브로커가 자동 관리

### __consumer_offsets

- 카프카 브로커 내부에 자동으로 생성되는 내부 토픽(Internal Topic)
- 컨슈머 그룹이 어떤 토픽의 어떤 파티션을 어디까지 읽었는지(오프셋 위치)를 저장한다.

<br>

## 3.5 카프카 스트림즈

- 카프카 공식 라이브러리
- 토픽에 적재된 데이터를 실시간으로 변환 → 가공 후 다시 다른 토픽에 적재
- 애플리케이션 코드에서 직접 실행 가능

즉, 카프카 데이터를 SQL 없이 코드로 실시간 ETL(Extract-Transform-Load) 하는 도구

### 태스크

- 스트림즈 애플리케이션의 데이터 처리 최소 단위
- 입력 토픽의 파티션 수와 동일하게 태스크가 생성됨

<img width="653" height="263" alt="Image" src="https://github.com/user-attachments/assets/39fd869f-29fa-4c23-ac35-18a445357086" />

### 토플로지(Topology)

카프카 스트림즈의 구조와 사용방법을 알기 위해서는 우선 토플로지(topology)와 관련된 개념을 익혀야 한다.

- 스트림즈 애플리케이션에서 정의한 데이터 처리 흐름도
  - 노드(Node): 데이터 처리 단계 (프로세서) 
  - 선(Edge): 데이터 흐름(스트림)
- 트리(Tree)와 유사한 구조

<img width="639" height="340" alt="Image" src="https://github.com/user-attachments/assets/120c7134-6d94-420f-b59f-c78f6ace7355" />

### 프로세서

토폴로지를 이루는 노드는 프로세서(Processor) 라고 부른다.

- 소스 프로세서 (Source Processor)
  - 데이터가 들어오는 최초 노드 
  - 특정 토픽에서 레코드를 읽음 
- 스트림 프로세서 (Stream Processor)
  - 실제 데이터 변환/처리 로직 수행 
  - 예: 필터링, 매핑, 분기, 집계
- 싱크 프로세서 (Sink Processor)
  - 처리된 데이터를 최종적으로 카프카 토픽에 적재

### 스트림즈DSL

스트림즈 DSL에는 레코드의 흐름을 추상화한 3가지 개념인 KStream, KTable, GlobalKTable이 있다.

#### KStream

- 정의: 토픽의 레코드 하나하나를 “이벤트 스트림”으로 다룸.

- 특징:
  - 메시지 키와 값으로 구성된, 변화(이벤트)의 연속.
  - Consumer가 토픽을 구독하는 것과 거의 동일한 개념.
  - 예: order-created 토픽 → 주문이 들어올 때마다 하나씩 흘러가는 데이터.

<img width="646" height="428" alt="Image" src="https://github.com/user-attachments/assets/343b74aa-4778-4a5a-b272-02eb5f24f687" />

#### KTable

KTable: 데이터의 "상태"

- 정의: 같은 메시지 키를 기준으로 최신값만 유지하는 테이블.
- 특징:
  - 키가 동일하면 이전 값은 덮어쓰기(업데이트).
  - 예: address-table 토픽 → 특정 고객 ID의 주소가 바뀌면 최신 주소만 저장.

<img width="651" height="427" alt="Image" src="https://github.com/user-attachments/assets/a3a376d7-cedb-4f82-855b-33340dc671a8" />

#### GlobalKTable

GlobalKTable: "전체 복제된 상태"

- 정의: KTable과 동일하지만, 모든 파티션의 데이터가 모든 태스크에 복제됨. 
- 특징:
  - 각 태스크가 토픽의 전체 데이터를 로컬에 가짐.
  - 단점: 모든 태스크가 데이터를 다 들고 있으므로 로컬 스토리지 사용량 증가.

KTable은
- 태스크 1 → 파티션 0 담당
- 태스크 2 → 파티션 1 담당
- 태스크 3 → 파티션 2 담당

GlobalKTable로 선언하면 → 태스크 1, 태스크 2, 태스크 3 모두 파티션 0,1,2 전체 데이터를 가짐.

<img width="640" height="324" alt="Image" src="https://github.com/user-attachments/assets/c68c0d8a-a7c7-4b2b-a1aa-b10996f94d02" />

#### 코파티셔닝과 리파티셔닝

- 문제: KStream과 KTable을 조인하려면, 두 토픽이 같은 파티션 개수와 같은 파티셔닝 전략을 가져야 함.
- 코파티셔닝 불일치 시:
  - 조인이 불가능 → TopologyException 발생. 
- 해결책:
  - 리파티셔닝: 새로운 토픽을 만들어 키를 다시 배치해서 파티션 정렬. 
  - GlobalKTable 사용: 코파티셔닝 상관없이 조인 가능 (단, 메모리 많이 사용).


<img width="639" height="347" alt="Image" src="https://github.com/user-attachments/assets/1cb45b9b-8457-4be8-a21c-56b7a59a3953" />

조인을 수행하는 KStream과 KTable이 코파티셔닝되어 있지 않으면 KStream 또는 KTable을 리파티셔닝 하는 과정을 거쳐야 한다.

#### KStream, KTable, GlobalKTable

- 토픽 자체는 동일하다. 즉, KStream, KTable, GlobalKTable은 토픽 생성 방식의 차이가 아니라 DSL에서 바라보는 관점의 차이이다. 
- 따라서 코드에서 이렇게 선언하는 순간:

```java
KTable<String, String> addressTable = builder.table(ADDRESS_TABLE);
KStream<String, String> orderStream = builder.stream(ORDER_STREAM);
```

- ADDRESS_TABLE 토픽 → 상태(최신값) 중심으로 다룸.
- ORDER_STREAM 토픽 → 이벤트 흐름 중심으로 다룸.

즉, 같은 토픽이라도 DSL에서 stream()으로 선언하면 KStream, table()로 선언하면 KTable로 동작한다.




### 헷갈렸던 내용

#### 컨슈머 그룹 관련 파티션 읽을때
- 그룹 내: 파티션 1개는 최대 1개의 컨슈머에만 할당 (중복 없음).
- 그룹 간: 서로 다른 그룹이라면 파티션 제한 없이 동시에 읽기 가능 (중복 소비).
  - 그룹지정 안한것도 포함

#### kafka Exactly-once 보장하는방법

1) 일반 프로듀서/컨슈머로 EOS 구현

- 프로듀서
  - enable.idempotence=true
  - transactional.id=your-tx-id ← 트랜잭션 프로듀서 활성화 핵심
  - (암묵적으로) acks=all, retries>0, max.in.flight.requests.per.connection<=5 적용/권장

- 컨슈머
  - isolation.level=read_committed ← 커밋된 레코드만 읽음
  - enable.auto.commit=false 권장 (오프셋은 프로듀서 트랜잭션에 포함)

- 코드 흐름
  1. producer.beginTransaction()
  2. 레코드 처리 후 결과 producer.send(...)
  3. producer.sendOffsetsToTransaction(currentOffsets, groupMeta) ← 읽은 위치를 같은 트랜잭션에 포함
  4. producer.commitTransaction()

2) Kafka Streams 사용 시(가장 간단)
```java
processing.guarantee=exactly_once_v2
```
- 내부적으로 멱등성 + 트랜잭션 + 오프셋 동시 커밋을 자동으로 처리합니다.



