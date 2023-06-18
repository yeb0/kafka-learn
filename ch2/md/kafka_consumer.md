# Kafka

## consumer

해당 topic으로 전송한 레코드(데이터)는 console-consumer 명령어로 확인할 수 있다.

```
kafka-console-consumer.bat 
--bootstrap-server localhost:9092 
--topic hello2 
--from-beginning
```

데이터 메시지 키와 값을 보고 싶다면 --property 옵션을 이용하면 된다.

```
kafka-console-consumer.bat 
--bootstrap-server localhost:9092 
--topic hello2 
--property print.key=true 
--property key.separator="-" 
--group hello-group 
--from-beginning
```

- print.key는 메시지 키를 확인하기 위함으로, false일 시 볼 수 없다.
- separator="-" 로 지정해 두었는데 key-value 형식으로 나타나게끔 지정해둔 것이다. 기본으로는 key\value 형식이다.
- hello-group 이란 consumer group을 생성한다. 1개 이상의 컨슈머로 이루어져 있으며<br>
이 그룹을 통해 가져간 topic의 메시지는 가져간 메시지에 대해 commit 한다. commit은 특정 레코드까지 처리했다고 레코드의 offset 번호를
kafka broker에 저장하는 것. commit 정보는 consumer_offsets 이름의 내부 topic에 저장된다.

해당 그룹의 describe 명령어를 통해 상세정보를 볼 수 있다.
```
kafka-consumer-groups.bat --bootstrap-server localhost:9092 --group hello-group --describe

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST
   CLIENT-ID
hello-group     hello2          3          6               6               0               -               -
   -
hello-group     hello2          2          7               7               0               -               -
   -
hello-group     hello2          1          3               3               0               -               -
   -
hello-group     hello2          0          6               6               0               -               -
```

아래는 kafka를 AWS로 실습하지 않아서, 실습을 진행하지는 못했으나 네트워크 통신 간단히 할 때에 쓰이기에 가져왔다.


>kafka-verifiable-producer, consumer.sh Permalink
kafka-verifiable로 시작하는 2개의 스크립트를 사용하면 String 타입 메시지 값을 코드 없이 주고받을 수 있습니다.
카프카 클러스터 설치가 완료된 이후에 토픽에 데이터를 전송하여 간단한 네트워크 통신 테스트를 할 때 유용합니다.
```
bin/kafka-verifiable-producer.sh --bootstrap-server my-kafka:9092 --max-messages 10 --topic verify-test
```
```
# 출력
# 최초 실행 시점
{"timestamp":1646221251926,"name":"startup_complete"} 
# 메시지별로 보낸 시간과 메시지 키, 메시지 값, 토픽, 저장된 파티션, 저장된 오프셋 번호
{"timestamp":1646221252222,"name":"producer_send_success","key":null,"value":"0","offset":0,"topic":"verify-test","partition":0}
``` 

```
# 10개의 데이터가 모두 전송된 이후 통계값 출력 - 평균 처리량 확인
{"timestamp":1646221252236,"name":"tool_data","sent":10,"acked":10,"target_throughput":-1,"avg_throughput":32.15434083601286}
```

- bootstrap-server : 통신하고자 하는 클러스터 호스트와 포트 입력
- max-messages : kafka-verifiable-producer.sh로 보내는 데이터 개수를 지정한다.<br>-1을 옵션값으로 입력하면 producer가 종료될 때까지 계속 데이터를 토픽으로 보낸다.
- topic : 데이터를 받을 대상 토픽 지정

이제 전송한 데이터를 kafka-verifiable-consumer.sh로 확인해 봅시다.
```
bin/kafka-verifiable-consumer.sh --bootstrap-server my-kafka:9092 --topic verify-test --group-id test-group
```
```
# 출력
# 시작
{"timestamp":1646221635415,"name":"startup_complete"}
# 컨슈머는 토픽에서 데이터를 가져오기 위해 파티션에 할당하는 과정을 거치는데 0 파티션이 할당된 것을 확인
{"timestamp":1646221635851,"name":"partitions_assigned","partitions":[{"topic":"verify-test","partition":0}]}
{"timestamp":1646221635938,"name":"records_consumed","count":10,"partitions":[{"topic":"verify-test","partition":0,"count":10,"minOffset":0,"maxOffset":9}]}
# 컨슈머는 한 번에 다수의 메시지를 가져와 처리하므로 한 번에 10개의 메시지를 정상적으로 받음을 확인
# 메시지 수신 이후 10번 오프셋 커밋 여부도 확인 가능, 정상적으로 커밋 완료
{"timestamp":1646221635949,"name":"offsets_committed","offsets":[{"topic":"verify-test","partition":0,"offset":10}],"success":true}
```

- bootstrap-server : 통신하고자 하는 클러스터 호스트와 포트 입력
- topic : 데이터를 가져오고자 하는 토픽 지정
- group-id : 컨슈머 그룹 지정

### delete
이미 적재된 topic의 data를 지우는 방식이다. kafka-delete-record를 사용한다.

kafka-delete-record는 이미 적재된 topic의 데이터 중 가장 오래된 데이터(가장 낮은 숫자의 offset 번호)부터 특정 시점까지 (offset까지)삭제할 수 있다. 

예를 들어, 0번부터 100번까지 있지만 0부터 30까지의 데이터를 지우고 싶다면...

```
vi delete-topic.json
{"partitions": 
    [{"topic":"test", "partition": 0, "offset": 30}], "version":1 
}

bin/kafka-delete.records.bat 
--bootstrap-server my-kafka:9092 
--offset-json-file delete-topic.json
```
- 삭제하고자 하는 데이터에 대한 정보를 파일로 지정해서 사용한다. 
- 해당 파일에는 삭제하고자 하는 토픽, 파티션, 오프셋 정보가 들어가야 한다.
- offset-json-file 옵션으로 삭제 정보를 담은 delete-topic.json을 입력하면 파일을 읽어서 데이터를 삭제한다.

주의할 점은 토픽의 특정 레코드 하나만 삭제하는 것이 아니라 **파티션에 존재하는 가장 오래된 오프셋부터 지정한 오프셋까지 삭제된다는 점**이다.

kafka에서 토픽의 파티션에 저장된 **특정 데이터만 삭제할 수는 없다.**

