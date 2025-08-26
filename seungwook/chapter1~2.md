
### 주키퍼 실행

```shell
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

### 카프카 실행

```shell
bin/kafka-server-start.sh -daemon config/server.properties
```

### 토픽 생성

```shell
bin/kafka-topics.sh --create --bootstrap-server ec2-3-39-24-240.ap-northeast-2.compute.amazonaws.com:9092 --topic hello.kafka
```

- 예전(카프카 2.1 이하)에는 --zookeeper 옵션을 써서 Zookeeper에 직접 연결해서 토픽 생성/삭제 같은 명령을 실행했다.

- 하지만 카프카 2.2 버전부터는 Zookeeper에 직접 붙는 대신, Kafka 브로커를 통해 이런 명령을 실행하도록 변경되었다.

- 이유: Zookeeper와 직접 통신하면 아키텍처 복잡도가 올라가기 때문.

- 따라서 카프카 2.5 이하 버전을 쓰더라도 토픽 관련 명령은 반드시 --bootstrap-server 옵션을 통해 Kafka 브로커에 접속해서 실행해야 한다.


> 토픽의 파티션을 늘릴수는 있지만 줄일수는 없다.


### 토픽 producer

```shell
bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka
```


### 토픽 consumer

```shell
bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning
```

프로듀서가 토픽에 넣은 순서와 컨슈머가 토픽에서 가져간 데이터의 순서가 달라지게 된다.

만약 토픽에 넣은 데이터의 순서를 보장하고 싶다면 가장 좋은방법은 파티션 1개로 구성된 토픽을 만드는것이다.
