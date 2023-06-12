# Kafka

자세한 내용들과 개념정리 같은 것들은 따로 정리하지 않고, 명령어나 사용법 위주로 작성할 것

[아파치 카프카 애플리케이션 프로그래밍](https://www.yes24.com/Product/Goods/99122569)을 보고 사용법을 익히는 중입니다.

## topic 
### 토픽을 생성할 수 있다.
kafka-topics.bat이 있는 디렉토리(bin\windows 하위)에서 cmd를 실행시켜 명령어를 사용하면 된다.

현재 환경은 localhost 환경이라 server 뒤에 localhost라 명시해둔다.
```
kafka-topics.bat 
--create 
--bootstrap-server localhost:9092 
--topic hello
```
hello라는 이름의 topic이 생성된다. create의 명령어와 함께 server port도 명시해 주어야 한다.

```
kafka-topics.bat 
--create 
--bootstrap-server localhost:9092 
--partitions 3 
--replication-factor 1 
--config retention.ms=172800000 
--topic hello2
```
partitions, replication-factor, config retention.ms 설정도 같이 넣어줄 수 있다.

- partitions : 파티션의 최소 개수를 정할 수 있다. 최소 개수는 1개이다.
- replication-factor : 토픽의 파티션을 복제할 복제 개수를 적는다. 1은 복제하지 않고 사용하겠다는 의미.
- config retention.ms : --config를 통해 kafka-topics.sh 명령에 포함되지 않은 추가적인 설정 가능 위의 예제는 topic 유지 기간이며, 2일을 뜻함. ms(밀리세컨드)단위


### 토픽들을 조회하기
생성하기와 마찬가지로 해당 명령어를 통해 현재 생성된 topic들을 조회할 수 있다.

```
kafka-topics.bat --bootstrap-server localhost:9092 --list
```
해당 topic에 대해서도 자세히 조회할 수 있다.
```
kafka-topics.bat --bootstrap-server localhost:9092 --describe --topic hello2
```

```
        Topic: hello2   Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: hello2   Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: hello2   Partition: 2    Leader: 0       Replicas: 0     Isr: 0
```
partitions와 leader(파티션의 리더(어디에 있는지?)), Replica(복제된 파티션이 어디에 존재?).. 볼 수 있다. 현쟈 leader가 0으로 되어 있는데

여러 대의 브로커로 카프카 클러스터를 운영할 때 토픽의 파티션이 몰려 있을 수 있는데 이 때 특정 리더에 여러 브로커가 다수 모여있을 경우엔

네트워크 대역 이슈가 생길 우려가 있다. 따라서 카프카 클러스터의 성능이 좋지 못할 경우에 한번씩 해당 명령어를 통해 확인하는 것도 좋다.

### 해당 토픽의 수정
topic을 수정할 수 있다.

partition의 개수와 유지기간을 수정할 수 있다. 하지만 partitions의 개수는 줄일 수는 없고 늘릴 수만 있어서 신중히 고려 후 선택해야만 한다.
- 왜 늘일 수만 있는데 신중히 고려해야하는가? <br>
물론 partition이 늘어남에 따라 처리할 수 있는 속도와 양을 얻어갈 수는 있으나, 많아짐에 따른 단점이 존재한다.

> 오히려 파티션이 많아지면 노드(브로커)가 다운이 됐을 경우 관리해야 될 포인트가 많아지고 오래 걸린다. 왜냐하면 내부적으로 토픽단위로 관리하는 것이 아니라 토픽안에 있는 파티션 단위로 분산 처리를 하기 때문이다.

Kafka의 Replication은 partition 기준으로 돌기에 partition이 많을 경우 장애 복구의 시간이 그만큼 증가한다.

> 그리고 파티션이 많아지면 그만큼 파일 핸들러도 늘어나게 되어서 파티션이 많다고 무조건 좋다 볼 수 없다. 파티션은 필요한 만큼만 늘리는 것이 좋은데 예를 들어 현재 파티션 개수가 원하는 데이터를 충분히 처리하지 못할 때 파티션을 늘리는 것이다.

Ref - [카프카의 topic Partition은 많으면 좋은 것일까?](https://needjarvis.tistory.com/603)

지금까지 kafka-topics.bat 명령어를 이용해 사용했지만, 이번에 수정에서는 kafka-configs.bat 파일도 이용해서 사용해야 한다.

partition의 개수를 변경하려면 전자를 이용해야하고, 유지 일수(config retention.ms)는 configs를 이용해야 하기 때문이다.

- 설정이 파편화된 이유는 무엇인가? <br>
토픽에 대한 정보를 관리하는 일부 로직이 다른 명령어로 넘어갔기 때문이다.
```
kafka-topics.bat 
--bootstrap-server localhost:9092 
--topic hello2 
--alter 
--partitions 4

        Topic: hello2   Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: hello2   Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: hello2   Partition: 2    Leader: 0       Replicas: 0     Isr: 0
        Topic: hello2   Partition: 3    Leader: 0       Replicas: 0     Isr: 0
```
hello topic의 partition 개수를 4로 늘렸다. 
```
kafka-configs.bat 
--bootstrap-server localhost:9092 
--entity-type topics 
--entity-name hello2 
--alter 
--add-config retention.ms=86400000

Completed updating config for topic hello2.
----------

kafka-configs.bat 
--bootstrap-server localhost:9092 
--entity-type topics 
--entity-name hello2 
--describe
<-- 내용 -->
Dynamic configs for topic hello2 are:
  retention.ms=86400000 sensitive=false synonyms={DYNAMIC_TOPIC_CONFIG:retention.ms=86400000}
```

partitions와 config 파일을 이용해 유지 기간을 수정하였다.
