# Kafka 

producer 환경에 따라 특정 data를 가지는 record를 특정 partition에 보내야할 때가 있다. 

0번 partition에 어떠한 값을 가진 메시지 키가 들어가야한다면 기본으로 설정했을 때에는 key의 해시값을 partition에 매칭하여 데이터를 전송하니
어느 partition으로 들어가는지 알 수 없다.

Partitioner interface를 사용하여 custom partitioner를 생성하면 사용자가 원하는 대로 지정하여 넣을 수 있다.

```java
package org.example;

import java.util.List;
import java.util.Map;
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.InvalidRecordException;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.utils.Utils;

public class CustomPartitioner implements Partitioner {

  @Override
  public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes,
      Cluster cluster) {
    if (keyBytes == null) {
      throw new InvalidRecordException("Need message key");
    }
    if (key.equals("Pangyo")) { // key is Pangyo일 경우
      return 0;
    }
    List<PartitionInfo> partitionInfos = cluster.partitionsForTopic(topic); // Pangyo가 아닌 다른 경우엔 hash값을 지정하여 특정 파티션에 매칭
    int numPartitions = partitionInfos.size();
    return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
  }

  @Override
  public void configure(Map<String, ?> configs) {}

  @Override
  public void close() {}
}

```

그리고 SimpleProducer class에 
```java
configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, CustomPartitioner.class);
```
추가해준다. 

Producer를 동기/비동기로 결과를 받을 수 있다. 기본적으로 동기의 결과이나, 비동기로 결과값을 받고 싶다면 Callback Interface를 이용하면 된다.

```java
package org.example;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 비동기로 결과를 확인할 수 있도록 하는 Callback Interface
 */
public class ProducerCallback implements Callback {

  private final static Logger logger = LoggerFactory.getLogger(ProducerCallback.class);

  @Override
  public void onCompletion(RecordMetadata metadata, Exception exception) {
    if (exception != null) {
      logger.error(exception.getMessage(), exception);
    } else {
      logger.info(metadata.toString());
    }
  }
}
```

SimpleProducer에는 
```java
   ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, messageValue); // 비동기
   producer.send(record, new ProducerCallback()); // 비동기 처리
```
추가함에 따라 비동기로 결과를 받을 수 있다. 순서는 보장되지 않으며, 동기로 결과를 받는 것보다 속도는 빠르다.

비동기로 결과를 기다리는 동안 다음으로 보낼 데이터의 전송이 성공하고 앞서 보낸 데이터의 결과가 실패할 경우, 순서가 역전될 가능성이 있다.

따라서 순서를 보장하고자 한다면 동기를, 아니라면 비동기를 선택해 프로젝트에 맞게 설정하는 것이 맞겠다.