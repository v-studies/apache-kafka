### 4.4 스프링 카프카 


#### 4.4.1 스프링 카프카 프로듀서 

```java
@Configuration
public class KafkaTemplateConfiguration {
	@Bean
	public KafkaTemplate<String, String> customKafkaTemplate(){
		Map<String, Object> props = new HashMap<>();
		props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
		props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
		props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

		ProducerFactory<String,String> producerFactory = new DefaultKafkaProducerFactory<>(props); 
		return new KafkaTemplate<>(producerFactory); //프로듀서 팩토리 클래스 통해서 카프카 템플릿 생성 
	}
}
```


```java
@SpringBootApplication
public class KafkaApplication implements CommandLineRunner {

	private final static String TOPIC_NAME = "test";

	@Autowired
	private KafkaTemplate<String, String> customKafkaTemplate;

	public static void main(String[] args) {
		SpringApplication.run(KafkaApplication.class, args);
	}

	@Override
	public void run(String... args) {
		CompletableFuture<SendResult<String, String>> future = customKafkaTemplate.send(TOPIC_NAME, "test"); //카프카 템플릿 클래스 사용하여 데이터 전송 

		try {
			SendResult<String, String> result = future.get();
			System.out.println("메시지 전송 성공: " + result.getRecordMetadata());
		} catch (Exception ex) {
			System.out.println("메시지 전송 실패: " + ex.getMessage());
		}
	}
}
```

#### 4.4.2 스프링 카프카 컨슈머 

acks vs ackMode

1. acks (Producer 설정)
- Kafka Producer에서 사용하는 설정으로, 메시지를 보낸 후 브로커로부터 어떤 수준의 확인을 받을지 결정

2. ackMode (Consumer 설정) --> 수동 커밋 설정
- Kafka Consumer (특히 Spring Kafka)에서 사용하는 설정으로, Consumer가 메시지를 처리한 후 언제 offset을 커밋할지 결정

** enable.auto.commit=true --> 카프카 컨슈머가 일정 주기로 메시지 오프셋을 자동으로 커밋하기 때문에, AckMode 설정은 무시
- 자동 커밋 활성화
- 일정 간격(auto.commit.interval.ms)마다 자동으로 offset 커밋


##### 4.4.2.1 레코드 리스터 
```java
     // poll()이 호출되어 가져온 레코드들은 차례대로 개별 레코드의 메시지 값을 파라미터로 받게 된다. 
	@KafkaListener(topics = "test", groupId = "test-group-00")
	public void recordLister(ConsumerRecord<String,String> record){
		log.info("record : {}", record.toString());
	}

    // 역직렬화 클래스 기본값인 StringDeserializer로 String 메세지 값을 전달 받게 된다. 
	@KafkaListener(topics = "test", groupId = "test-group-01")
	public void singleTopicListener(String messageValue){
		log.info("messageValue : {}", messageValue);
	}

```

- 2개 이상의 카프카 컨슈머 스레드를 실행하고 싶다면 concurrency 옵션 사용
- 특정 토픽의 특정 파티션만 구독하고 싶다면 topicPartitions 파라미터 사용

##### 4.4.2.2 배치 리스너
```java
	@KafkaListener(topics = "test", groupId = "test-group-02")
	public void consumerCommitListener(List<ConsumerRecord<String, String>> records){
		records.forEach(record -> log.info("record : {}", record));
	}
```
** 배치 리스너 활성화 필요 

<img width="515" height="28" alt="image" src="https://github.com/user-attachments/assets/9e15677a-8c15-4c29-87d1-1a693194608b" />

### 카프카 실전 프로젝트

##### 데이터 포맷
- JSON은 String으로 선언되어 스키마의 변경에 유연하게 대처할 수 있다.

##### 프로듀서
- akcs = 1 -> 최소한 리더 파티션에는 데이터 적재 -> min.insync.replicas 최소 동기화 리플리카 설정은 리더 파티션에 지속 적재하므로  acks =1 일 경우 무시된다. ack를 all로 설정할 경우에만 min.insync.replicas 설정이 유효하다.

