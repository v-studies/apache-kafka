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
