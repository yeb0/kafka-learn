# Kafka 

Java로 코드를 작성하고 프로젝트를 실행해 보면서 producer에 사용되는 class와 method를 익힐 수 있었다.

```java
package org.example;

import java.util.Properties;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


public class SimpleProducer {
  private final static Logger logger = LoggerFactory.getLogger(SimpleProducer.class);
  private final static String TOPIC_NAME = "test";
  private final static String BOOTSTRAP_SERVERS = "LOCALHOST:9092";

  public static void main(String[] args) {
    Properties configs = new Properties();
    configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
    configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

    KafkaProducer<String, String> producer = new KafkaProducer<>(configs);

    String messageValue = "testMessage";
    ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, messageValue);
    producer.send(record);
    logger.info("{}", record);
    producer.flush();
    producer.close();
  }
}
```
명령어 사용했던 것과 크게 다를 게 없었는데, 해당 topic에 messageValue(메시지)를 넣어주는 간단한 코드이다.
아래에는 결과 로그를 볼 수 있다.
```
16:03:31.241 [main] INFO org.apache.kafka.clients.producer.ProducerConfig - ProducerConfig values: 
	acks = 1
	batch.size = 16384
	bootstrap.servers = [LOCALHOST:9092]
	buffer.memory = 33554432
	client.dns.lookup = default
	client.id = producer-1
	compression.type = none
	connections.max.idle.ms = 540000
	delivery.timeout.ms = 120000
	enable.idempotence = false
	interceptor.classes = []
	...
생략

16:03:32.788 [kafka-producer-network-thread | producer-1] DEBUG org.apache.kafka.clients.producer.internals.Sender - [Producer clientId=producer-1] Starting Kafka producer I/O thread.
16:03:32.799 [main] INFO org.apache.kafka.common.utils.AppInfoParser - Kafka version: 2.5.0
16:03:32.802 [kafka-producer-network-thread | producer-1] DEBUG org.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Initialize connection to node LOCALHOST:9092 (id: -1 rack: null) for sending metadata request
16:03:32.802 [main] INFO org.apache.kafka.common.utils.AppInfoParser - Kafka commitId: 66563e712b0b9f84
16:03:32.802 [main] INFO org.apache.kafka.common.utils.AppInfoParser - Kafka startTimeMs: 1686726212787
16:03:33.232 [main] INFO org.example.SimpleProducer - ProducerRecord(topic=test, partition=null, headers=RecordHeaders(headers = [], isReadOnly = true), key=null, value=testMessage, timestamp=null)
... 생략
```
kafka의 버전과 전송한 ProducerRecord가 출력된다. ProducerRecord 인스턴스 생성 시 메시지 키를 설정하지 않아 null로 설정된 것을 확인할 수 있었다.

topic에 record가 적재되었는지 확인하기 위해 kafka-console-consumer 명령어를 통해서 확인할 수 있다.

이미 producer application을 실행했으니 --from-beginning 속성을 넣어 확인하면 topic에 있는 모든 record를 조회할 수 있다.

```
kafka-console-consumer.bat 
--bootstrap-server localhost:9092 
--topic test 
--from-beginning
<-- 결과 -->
testMessage
```

Producer Application을 실행할 때 설정해야 하는 필수 옵션과 선택 옵션이 있는데, 필수 옵션만 기재하도록 한다.
선택옵션이 그렇다고 중요하지 않다는 것이 아니니 언젠가 필요할 때쯤 p88 찾아볼 것

- bootstrap.servers : 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커의 호스트 이름 : port 1개이상 작성한다.2개 이상 브로커 정보를 입력하여 일부 브로커에 이슈가 발생하더라도 접속하는 데에 이슈가 없도록 설정 가능
- key.serializer : 레코드의 메시지 키를 직렬화하는 클래스를 지정한다.
- value.serializer : 레코드의 메시지 값을 직렬화하는 클래스를 지정한다.

저번 kafka_producer.md에서 적었던
```
kafka-console-consumer.bat 
--bootstrap-server localhost:9092 
--topic test 
--property print.key=true 
--property key.separator="-" 
--from-beginning
```
key, value를 볼 수 있게끔 해당 속성들을 두어 확인할 수 있었다. 위의 SimpleProducer class에서 
```
ProducerRecord<String, String> record = new ProducerRecord<>("test", "Pangyo", "23");
```
를 생성하여 run 후에 list를 살펴보면, 

```
null-testMessage
Pangyo-23
```
위와 같은 결과를 확인할 수 있다.

물론 파티션을 직접 지정하여 record를 보낼 수 있다. (오버로딩) 
