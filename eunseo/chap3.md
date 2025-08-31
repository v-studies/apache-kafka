## 카프카 브로커
- 카프카 클라이언트와 데이터를 주고받기 위해 사용하는 주체
- 데이터를 분산 저장하여 장애가 발생하더라도 안전하게 사용할 수 있도록 도와줌 

### 데이터 저장, 전송 
- 토픽 이름과 파티션 번호의 조합으로하위 디렉토리를 생성하여 데이터를 저장한다.
- 카프카는 데이터를 메모리나 데이터베이스가 아니라 파일에 저장한다. OS에서 페이지 캐시(page cache)를 사용해 디스크 입출력 속도를 높였다. (한 번 읽은 파일의 내용은 메모리의 페이지 캐시 영역에 저장된다.)

### 데이터 복제 replication
- 복제는 파티션 단위로 이루어진다.
- 토픽 생성 시 파티션의 복제 개수도 같이 설정한다. (replicaiton factor) 
- 복제된 파티션은 leader와 follower로 구성된다.
   - 리더: 프로듀서 또는 컨슈머와 직접 통신하는 파티션
   - 팔로워: 복제 데이터를 가지고 있는 파티션.  리더의 오프셋을 확인해서 자신의 오프셋과 차이 나는 경우 데이터를 가져온다. 

### 컨트롤러
- 클러스터의 여러 브로커 중 한 대가 컨트롤러 역할을 한다.
- 컨트롤러는 다른 브로커들의 상태를 체크하고 브로커가 클러스터에서 빠지는 경우 해당 브로커에 존재하는 리더 파티션을 재분배한다.


### 데이터 삭제
-  컨슈머가 데이터를 가져가도 토픽의 데이터가 삭제되지 않는다.
-  컨슈머나 프로듀서가 데이터 삭제를 요청할 수도 없다. 오직 브로커만이 데이터를 삭제할 수 있다.
- 데이터 삭제는 파일 단위, '로그 세그먼트' 단위로 이뤄진다. 

### 컨슈머 오프셋 저장
- 컨슈머 그룹은 토픽이 특정 파티션으로부터 데이터를 가져가서 처리하고 이 파티션의 어느 레코드까지 가져갔는지 확인을 위해 오프셋을 커밋한다.
- 커밋한 오프셋은 '__consumer_offsets' 토픽에 저장된다. 

### 코디네이터 coordinator
- 클러스터의 여러 브로커 중 한 대가 코디네이터 역할을 한다.
- 코디네이터는 컨슈머 그룹의 상태를 체크하고 파티션을 컨슈머와 매칭되도록 분배한다.
- 컨슈머가 컨슈머 그룹에서 빠지면 매칭되지 않은 파티션을 정상 동작하는 컨슈머로 할당하여 끊임없이 데이터가 처리되도록 도와준다. (=리밸런스 rebalance)
 
### 주키퍼 zeekeeper
- 카프카의 메타데이터를 관리한다. 

### 토픽과 파티션 
- 데이터(record)를 구분하기 위해 사용하는 단위
- 토픽은 1개 이상의 파티션을 소유한다.
- 파티션은 카프카의 병렬처리의 핵심으로 그룹으로 묶인 컨슈머들이 레코드를 병렬로 처리할 수 있도록 매칭한다.
- 컨슈머의 처리량이 한정된 상황에서 많은 레코드를 병렬로 처리하기 위한 가장 좋은 방법은 컨슈머의 개수를 늘리는 것이다. 컨슈머 개수를 늘리면서 동시에 파티션 개수도 늘리면 처리량을 증가시킬 수 있다.


```
컨슈머 그룹을 사용하지 않는 경우

각 컨슈머가 독립적으로 토픽의 모든 파티션을 읽을 수 있음
컨슈머만 늘리면 처리량 증가 가능
다만 중복 처리 발생 (같은 메시지를 여러 컨슈머가 처리)

컨슈머 그룹을 사용하는 경우
파티션과 컨슈머의 관계를 고려해야 합니다:
핵심 제약사항:

하나의 파티션은 같은 컨슈머 그룹 내에서 최대 하나의 컨슈머에만 할당
컨슈머 수 > 파티션 수인 경우, 일부 컨슈머는 유휴 상태

```

파티션 3, 그룹 컨슈머 멤버 1일 경우
컨슈머 멤버 1일 파티션 3개 모두 담당

<img width="1359" height="309" alt="image" src="https://github.com/user-attachments/assets/c099b3d8-127c-4d0d-89eb-995c28755c25" />

파티션 3, 그룹 컨슈머 멤버 2일 경우 --> 자동 리밸런싱 
컨슈머 멤버 1가 파티션 0,1을 담당
컨슈머 멤버 2가 파티션 2를 담당  
<img width="1366" height="342" alt="image" src="https://github.com/user-attachments/assets/d568398a-05ed-44ea-8d8d-07ebe819e5c7" />

파티션 2, 컨슈머 멤버 2 

<img width="395" height="169" alt="image" src="https://github.com/user-attachments/assets/2f2046da-09a6-4894-833d-f9a9b4521804" />
<img width="1360" height="324" alt="image" src="https://github.com/user-attachments/assets/26dfe631-e73f-4a44-8095-dbbe968f10f7" />
<img width="1367" height="284" alt="image" src="https://github.com/user-attachments/assets/fb2bfc91-7f8d-4845-8383-ce11463a4526" />



### 토픽 이름 제약 조건

| 예시 
```
<환경>.<팀명>.<애플리케이션명>.<메시지타입>
prd.marketing-team.sms-platform.json
<프로젝트명>.<서비스명>.<환경>.<이벤트명>
commerece.payment.prd.notification
<환경>.<서비스명>.<JIRA번호>.<메시지타입>
dev.email-sender.jira-1234.email-vo-custom
<카프카클러스터명>.<환경>.<서비스명>.<메시지타입>
aws-kafka.live.marketing-platform.json
```

### 레코드
- 동일 키라면 동일 파티션으로 들어감 (파티션 개수 변경 시 재분배)
- 메시지 키와 메시지 값은 직렬화되어 브로커로 전송되기 때문에 동일한 형태로 역직렬화를 수행해야 한다.
- 오프셋은 직접 지정할 수 없으며, 브로커에 저장될 때 이전에 전송된 레코드의 오프셋+1 값으로 생성된다.


### 프로듀서 설정 
```java
package com.example.kafka;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

/**
 * Kafka Producer 예제
 * 메시지를 Kafka 토픽으로 전송하는 간단한 Producer 구현
 */
@Slf4j
public class SimpleProducer {
	// 메시지를 전송할 토픽명
	private final static String TOPIC_NAME = "test";
	// Kafka 브로커 서버 주소
	private final static String BOOTSTRAP_SERVERS = "localhost:9092";

	public static void main(String[] args) {
		// Producer 설정
		Properties configs = new Properties();
		configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
		configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
		configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

		// Kafka Producer 생성
		KafkaProducer<String, String> producer = new KafkaProducer<>(configs);

		// 전송할 메시지 데이터
		String messageKey = "key1";
		String messageValue = "testMessage";
		
		// ProducerRecord 생성 (토픽, 키, 값)
		ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, messageKey, messageValue);
		
		// 메시지 전송
		producer.send(record);
		log.info("record : {}", record);
		
		// 버퍼에 남은 메시지 즉시 전송
		producer.flush();
		
		// Producer 종료
		producer.close();
	}
}
```

<img width="1360" height="637" alt="image" src="https://github.com/user-attachments/assets/cd897edd-2d0f-406c-832d-824a48c49cd8" />

<img width="564" height="125" alt="image" src="https://github.com/user-attachments/assets/7309ece7-87b3-495c-82c2-0945d2498812" />


| 카프카 파티션 정책
```
- Kafka 2.4.0부터 UniformStickyPartitioner가 기본 정책이다. 
- 배치가 가득 찰 때까지 같은 파티션에 고정하여 네트워크 요청을 줄이므로 RoundRobinPartitioner보다 성능이 좋다.
- 커스텀 파티션 정책은 Partitioner 인터페이스를 구현하여 사용할 수 있다.
- 메세지 키가 있을 때는 키의 해시값과 파티션을 매칭하여 전송한다. --> 키가 다양하면 여러 파티션으로 분산되어 배치가 작아질 수 있음 --> UniformStickyPartitioner의 배치 최적화 효과가 제한됨
```


| 프로듀서 설정 필수 옵션
```
bootstrap.servers: 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커의 '호스트 이름:포트'를 1개 이상 작성한다. 2개 이상 입력하여 일부 브로커에 이슈가 발생하더라도 접속 이슈가 없게 설정 가능하다.
key.serializer: 레코드의 메시지 키를 직렬화하는 클래스를 지정한다. 
value.serializer: 레코드의 메시지 값을 직렬화하는 클래스를 지정한다. 
```


