# Kafka

Producer가 전송한 record는 브로커에 적재된다. Consumer는 적재된 데이터를 사용하기 위해
브로커로부터 데이터를 가져와서 처리한다. 

```java
package org.example;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SimpleConsumer {
  private final static Logger logger = LoggerFactory.getLogger(SimpleConsumer.class);
  private final static String TOPIC_NAME = "test";
  private final static String BOOTSTRAP_SERVERS = "LOCALHOST:9092";
  private final static String GROUP_ID = "test-group";

  public static void main(String[] args) {
    Properties configs = new Properties();
    configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
    configs.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
    configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);
    consumer.subscribe(Arrays.asList(TOPIC_NAME));

    while(true) {
      ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
      for (ConsumerRecord<String, String> record : records) {
        logger.info("{}", record);
      }
    }
  }
}

```
while문을 돌면서 1초씩 for loop를 통해 poll() 메서드가 반환한 ConsumerRecord 값들을 순차적으로 처리한다.

```
kafka-console-producer.bat --bootstrap-server localhost:9092 --topic test
>testMessage
```

test topic에 testMessage라는 문자열의 데이터를 넣어주었을 때 출력화면에는 아래와 같은 값을 출력해 준다.

```
20:28:22.379 [main] INFO org.example.SimpleConsumer - ConsumerRecord(topic = test, partition = 1, leaderEpoch = 0, offset = 2, CreateTime = 1686742101338, serialized key size = -1, serialized value size = 11, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = testMessage)
```

마찬가지로 Consumer에도 Producer와 같이 필수/선택이 있으며, 필수는 Producer와도 같기에 따로 적지 않는다.
