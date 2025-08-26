# 아파치 카프카(Apache Kafka)

|  자바 객체를 그대로 전달할 수 있고, 서버에 분산되어 데이터가 파일 시스템에 안전하게 저장된다.


### 1. 높은 처리량
데이터를 여러 파티션에 분배해 병렬 처리하고, 파티션 수만큼 컨슈머를 늘려 동일 시간당 처리량을 높인다.
 

### 2. 확장성
가변적인 데이터량에 따라 브로커 수를 줄이거나 늘려 안정적으로 확장(스케일 인/아웃) 가능하다.

### 3. 영속성
전송받은 데이터를 파일 시스템에 저장해 영속성 보장한다. 
 

### 4. 고가용성
토픽생성할 때 min.insync.replicas 옵션을 2로 설정할 경우, 최소 2개 이상의 브로커에 데이터가 완벽히 복제됨을 보장한다. 
안전한 운영을 위해 카프카는 최소 3대 이상의 브로커 클러스터 구성이 권장된다.
****
#### 카프카 설정 
- 파티션 수 : 병렬처리 가능
- retention.ms : 카프카 브로커가 저장한 파일이 삭제되기까지 걸리는 시간--> -1은 영원히 삭제 x

| Docker Image Configuration Reference for Confluent Platform

https://docs.confluent.io/platform/current/installation/docker/config-reference.html#confluent-ak-configuration

<img width="2300" height="160" alt="image" src="https://github.com/user-attachments/assets/54460c26-3633-435f-87e1-33816639b87f" />

 실행 중인 컨테이너 확인 후 직접 접속:
 
  | docker ps
  | docker exec -it <kafka_container_name> bash

  접속 후 토픽 생성:
  
  kafka-topics --create --topic kafka_topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

  토픽 목록 확인:
  
  kafka-topics --list --bootstrap-server localhost:9092

<img width="1006" height="60" alt="image" src="https://github.com/user-attachments/assets/0a2f4ffa-320d-43c7-8e87-1ffab1ee2ca6" />


producer 실행:

kafka-console-producer --topic kafka_topic --bootstrap-server localhost:9092


consumer 실행 :

- 처음부터 모든 메시지 읽기:
    - kafka-console-consumer --topic kafka_topic --bootstrap-server localhost:9092 --from-beginning

- 새로 들어오는 메시지만 읽기:
    - kafka-console-consumer --topic kafka_topic --bootstrap-server localhost:9092

 - 특정 파티션에서 읽기:
    - kafka-console-consumer --topic kafka_topic --bootstrap-server localhost:9092 --partition 0



 —> 옵션 설정한대로(retention.ms) 데이터는 사라지지 않지만 offset이 읽은 만큼 증가함

->  카프카에서 파티션 개수를 늘리면, 새로 프로듀싱되는 레코드의 파티션 배정 방식이 달라질 수 있다.

메시지 키가 없는 경우: 라운드 로빈 등 프로듀서의 기본 전략에 따라 새로운 파티션에도 균등하게 분배된다.

메시지 키가 있는 경우: 키를 해시(hash)하여 파티션을 선택하는데, 파티션 수가 변경되면 해시 결과가 달라져 기존 키-파티션 매핑이 깨진다.
→ 즉, 이전에 키가 0번 파티션에 들어갔다고 해도, 파티션 확장 후에는 동일 키가 반드시 0번으로 간다는 보장이 없다.


** 확인
- key : value 로 넣었을 때 동일 파티션에 들어감 

kafka-console-producer --topic kafka_topic --bootstrap-server localhost:9092 --property "parse.key=true" --property "key.separator=:"
<img width="1478" height="1254" alt="image" src="https://github.com/user-attachments/assets/a055f3ff-2a67-4662-81ed-a512abc00330" />


파티션 늘렸을 때 
<img width="2732" height="540" alt="image" src="https://github.com/user-attachments/assets/a9a68689-b3fa-42be-98bb-10942fb4c4ec" />

