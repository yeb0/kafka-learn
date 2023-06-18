# Kafka

Producer와 Consumer에 대해 간단히 알아보고 정리하고자 작성한다. 대략적인 아키텍처 구성과 동작을 살펴보자.

## Kafka의 특징에 대해서

- PUB/SUB ? <br>
데이터 Queue를 두고 서로 독립적으로 데이터를 생산, 소비한다.서로 의존성이 없어 안정적이게 데이터를 처리할 수 있게 된다.
- 고가용성과 확장성 ? <br>
Kafka는 cluster로 동작한다. 분산 처리를 통해 빠른 data 처리를 가능케 한다. server를 수평적으로 늘려 scale-out이 가능하다.
- Disk 순차 저장과 처리 <br>
메시지를 메모리 Queue에 적재하는 기존 메시지 시스템과 다르게 Kafka는 Disk에 순차적으로 저장한다. <br>
서버에 장애가 나도 메시지가 Disk에 저장되어 있어 데이터의 유실 걱정이 없다. <br>
Disk가 순차적으로 저장되어 있으므로 Disk I/O가 줄어들어 성능적으로 빨라짐에 효과가 있다.
- 분산처리 <br>
Kafka는 Partition이란 개념을 도입해 여러 개의 Partition을 Server에 분산시켜 나누어 처리가 가능하다. 메시지를 상황에 맞춰 빠르게 처리할 수 있게 됐다.

## Kafka를 사용하는 이유는 ?

1. **병렬처리에 의한 Data 처리율 향상** : Kafka는 병렬로 처리함으로서 data(record)를 빠르고 효과적으로 처리할 수 있다. disk에 순차적으로 데이터를 적재하기 때문에 임의접근 방식보다 훨씬 더 빠르게 데이터를 저장한다.
2. **데이터 유실 방지** : Disk에 저장되기 때문에 혹시나 서버가 내려가거나 해도 데이터가 유실될 일이 없이 재시작하여 기존 데이터를 안정적으로 처리가 가능함.
3. **Clustering에 의한 고가용성** : scale-out이 가능해 system 확장이 용이하고, 어떤 Server가 죽어도 service 자체가 중단될 일 없이 system 운용 가능

## 주요 개념

- 프로듀서 (Producer) : 데이터를 발생시키고 Kafka Cluster에 적재하는 프로세스이다.
- 카프카 클러스터 (Kafka Cluster) : 카프카 서버로 이루어진 클러스터이다. 카프카 클러스터를 이루는 주 요소는 아래와 같다. <br>
1. **브로커 (Broker)** : 카프카 서버이다. <br>
2. **주키퍼 (Zookeeper)** : 주키퍼는 분산 코디네이션 시스템이다. 카프카 브로커를 하나의 클러스터로 코디네이팅하는 역할을 하며 카프카 클러스터의 리더(Leader)를 발탁하는 방식도 주키퍼가 제공하는 기능 이용 <br>
3. **토픽 (Topic)** : 카프카 클러스터에 데이터를 관리할 시 기준이 되는 개념. 토픽은 카프카 클러스터에서 여러 개 만들 수 있고, 하나의 토픽은 1개 이상의 Partition를 갖고 있다.(여러 개 가능) 데이터를 관리하는 그룹이다. 예를 들면, 고속도로 게이트.
4. **파티션 (Partition)** : 토픽 당 데이터를 분산 처리하는 단위이다. Kafka에서는 토픽 안에 파티션을 나누어 그 수대로 데이터를 분산 처리한다. 카프카 옵션에서 지정한 replica의 수만큼 Partition이 각 server들에게 복제된다. 예를 들면 고속도로 게이트의 여러 관문들..?
5. **리더, 팔로워 (Leader, Follower)** : 카프카에서는 각 파티션 당 복제된 파티션 중에서 하나의 리더가 된다. Leader는 모든 읽기, 쓰기 연산 담당하게 된다. 나머지는 Follower가 되며 단순 리더의 데이터를 복사만 하는 일을 담당한다.
- 컨슈머 그룹 (Consumer Group) : 컨슈머의 집합을 구성하는 단위이다. 카프카에서는 컨슈머 그룹으로서 데이터를 처리하고, 컨슈머 그룹 안의 컨슈머 수만큼 Partition의 Data를 분산 처리하게 된다.

다른 Kafka md file에서도 언급했지만 **Partition은 늘릴 수는 있으나 줄일 수는 없다.** 늘리는 것에 대해서는 꼭 신중히 선택해야만 한다.

Kafka Cluster에서 record를 가져올 때 컨슈머 그룹(Consumer Group) 단위로 가져온다. 자신이 가져와야 하는 topic안의 Partition의 data를 Pull하게 되며 각각 Consumer Group안의 Consumer들이 Partition이 나뉘어져 있는 만큼 데이터 처리한다.

Kafka에서 Producer는 순차적으로 저장된 데이터 뒤에 붙이는 append 형식으로 write 연산을 진행하게 된다. Partition들은 각각의 데이터들을 순차적인 집합인 offset으로 구성돼 있다.

Consumer들은 Consumer Group으로 나뉘어서 데이터를 분산처리하는데, 같은 Consumer Group내에 있는 Consumer들끼리 같은 Partition의 Record를 처리할 수 없다.