
# 3장 카프카 기본 개념 설명

## 3.1 카프카 브로커, 클러스터, 주키퍼

### 데이터 복제, 싱크

카프카의 데이터 복제는 파티션 단위로 이루어진다.

<img width="640" height="292" alt="Image" src="https://github.com/user-attachments/assets/9f1532ee-0451-40e7-a419-a25913741de6" />

팔로워 파티션들은 리더 파티션의 오프셋을 확인하여 현재 자신이 가지고 있는 오프셋과 차이가 나는 경우 리더 파티션으로부터 데이터를 가져와서 자신의 파티션에 저장하는데, 이 과정을 복제라고 부른다.

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

파티션에는 프로듀서가 보낸 데이터들이 들어가 저장되는데 이 데이터를 레코드라고 부른다.

- 파티션은 자료구조에서 접하는 큐와 비슷한 구조라고 생각하면 쉽다.
- FIFO 구조와 같이 먼저 들어간 레코드는 컨슈머가 먼저 가져가게 된다.

<br>

## 3.3 레코드

- 레코드는 카프카의 데이터 단위 (타임스탬프, 키, 값, 오프셋, 헤더)
- 키 = 파티션 분배 기준 (같은 키는 같은 파티션 → 순서 보장)
- 오프셋 = 파티션 내 고유 번호, 브로커가 자동 관리

### __consumer_offsets

- 카프카 브로커 내부에 자동으로 생성되는 내부 토픽(Internal Topic)
- 컨슈머 그룹이 어떤 토픽의 어떤 파티션을 어디까지 읽었는지(오프셋 위치)를 저장한다.
- 기본적으로 __consumer_offsets 토픽은 여러 파티션으로 나뉘어 분산 저장된다.


<br>

## 3.4 카프카 클라이언트

<img width="654" height="275" alt="Image" src="https://github.com/user-attachments/assets/ad36e249-8b84-4f1c-ae34-da7be7400bf0" />

3개의 파티션을 가진 토픽을 효과적으로 처리하기 위해서는 3개 이하의 컨슈머로 이루어진 컨슈머 그룹으로 운영해야 한다.

컨슈머 그룹은 다른 컨슈머 그룹과 격리되는 특징을 가지고 있다.

따라서 카프카 프로듀서가 보낸 데이터를 각기 다른 역할을 하는 컨슈머 그룹끼리 영향을 받지 않게 처리할 수 있다는 장점을 가진다.

<img width="650" height="352" alt="Image" src="https://github.com/user-attachments/assets/03d3ddbd-3910-40db-8ed1-0caee040612b" />

<br>

## 3.5 카프카 스트림즈

- 카프카 공식 라이브러리.
- 토픽에 적재된 데이터를 실시간으로 변환 → 가공 후 다시 다른 토픽에 적재.
- 별도 클러스터가 필요 없음 → 애플리케이션 코드에서 직접 실행 가능.

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
  - 결국 토픽의 모든 레코드 중에서 **“최신 스냅샷”**을 반영하는 구조. 
  - 예: address-table 토픽 → 특정 고객 ID의 주소가 바뀌면 최신 주소만 저장.

<img width="651" height="427" alt="Image" src="https://github.com/user-attachments/assets/a3a376d7-cedb-4f82-855b-33340dc671a8" />

#### GlobalKTable

GlobalKTable: "전체 복제된 상태"

- 정의: KTable과 동일하지만, 모든 파티션의 데이터가 모든 태스크에 복제됨. 
- 특징:
  - 각 태스크가 토픽의 전체 데이터를 로컬에 가짐. 
  - 따라서 KStream과 조인할 때 코파티셔닝(co-partitioning) 필요 없음. 
  - 단점: 모든 태스크가 데이터를 다 들고 있으므로 로컬 스토리지 사용량 증가. 
- 사용 예시:
  - orderStream(주문 스트림)과 addressTable(주소 테이블)을 조인한다고 가정. 
  - KTable로는 파티션 전략이 맞아야 조인이 가능하지만, GlobalKTable이면 그냥 모든 태스크에서 전체 주소 데이터를 들고 있으므로 조인 가능.


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

> 리파티셔닝이란 새로운 토픽에 새로운 메시지 키를 가지도록 재배열하는 과정이다.
> 
> 리파티셔닝 과정을 거쳐 KStream 토픽과 KTable로 사용하는 토픽이 코파티셔닝되도록 할 수 있다.

<img width="654" height="300" alt="Image" src="https://github.com/user-attachments/assets/6719549d-651c-4089-a081-ac0de8fae453" />


- 토픽 자체는 동일하다.. 즉, KStream, KTable, GlobalKTable은 토픽 생성 방식의 차이가 아니라 DSL에서 바라보는 관점의 차이이다. 
- 따라서 코드에서 이렇게 선언하는 순간:

```java
KTable<String, String> addressTable = builder.table(ADDRESS_TABLE);
KStream<String, String> orderStream = builder.stream(ORDER_STREAM);
```

- ADDRESS_TABLE 토픽 → 상태(최신값) 중심으로 다룸.
- ORDER_STREAM 토픽 → 이벤트 흐름 중심으로 다룸.

즉, 같은 토픽이라도 DSL에서 stream()으로 선언하면 KStream, table()로 선언하면 KTable로 동작한다.












