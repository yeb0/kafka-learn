# Kafka 

간단하게 실습을 위해 OS에 맞는(windows) Kafka를 설치했다. 

책에는 AWS EC2를 이용하여 Kafka를 사용하는데, 나는 EC2를 사용하지 않고 이미 구성되어 있는 Kafka를 사용하기 위해서이다.

## 시작하기

C 폴더에 넣어 Kafka 폴더에서 cmd를 실행시켜 zookeeper와 kafka를 순서대로 실행시키면 끝이다.

먼저 zookeeper를 실행시키는 명령어는
```
bin\windows\zookeeper-server-start.bat config\zookeeper.properties
```
이며, cmd를 실행시켜 해당 zookeeper가 실행이 잘 되었는지 확인도 할 수 있다.
```
# windows에서 2181포트가 실행되었는지 확인
netstat -na | findstr "2181"
```
실행이 잘 되었다면 Kafka를 실행시키면 된다.
```
bin\windows\kafka-server-start.bat config\server.properties
```
마찬가지로 확인하는 방법은
```
# windows에서 9092포트가 실행되었는지 확인
netstat -na | findstr "9092"
```
정상적으로 실행이 됐다면 성공이다.

![image](https://github.com/yeb0/kafka-learn/assets/119172260/a252095b-fc9f-4e91-a669-c7f5a84be067)

Ref - [windows에 Kafka 설치](https://herojoon-dev.tistory.com/118)
