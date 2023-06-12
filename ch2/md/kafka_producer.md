# Kafka

## producer
생성된 topic에 데이터를 넣을 수 있는 console-producer 명령어를 실행한다.

topic에 넣는 데이터는 '레코드'라 부르며, 메시지 키(key)와 값(value)로 이루어져 있다.

- key 없이 값만 보내는 작업을 실행해 보자.

```
kafka-console-producer.bat --bootstrap-server localhost:9092 --topic hello2
>hello
>kafka
>0
>1
>2
>3
```
주의할 점은 console-producer로 전송되는 레코드(데이터)의 값은 UTF-8을 기반으로
Byte로 변환되며 ByteArraySerializer로만 직렬화한다는 것이다. 

즉 String이 아닌 타입으로는 직렬화하여 전송할 수 없다. 텍스트 목적으로 문자열만 전송하며
다른 타입으로 직렬화하여 데이터를 브로커로 전송하고자 한다면 Kafka producer application을 직접
개발하여야 한다.

- key를 가지는 레코드(데이터)를 전송하기
```
kafka-console-producer.bat 
--bootstrap-server localhost:9092 
--topic hello2 
--property "parse.key=true" 
--property "key.separator=:"
>key1:no1
>key2:no2
>key3:no3
```
parse.key = true로 두게 된다면, 레코드를 전송할 때 메시지 key를 둘 수 있다.

자세한 내용은 p76 읽어볼 것