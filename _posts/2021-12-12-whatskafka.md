---
title: "Kafka 오리엔테이션"
excerpt_separator: "<!--more-->"
categories:
  - Kafka
tags:
  - Kafka
  - MessageQueue
sidebar:
  nav: "docs"
---
오랜만의 포스팅은 Kafka 로 시작해보고자 한다.  
신입사원 입사할 때 진행했던 오리엔테이션 내용을 간단히 정리해서 포스팅하는 것으로, 나도 오랜만에 기초를 다져보는 기회가 되지 않을까 생각한다.

## Apache Kafka
링크드인에서 개발한 대규모 분산 데이터 스트리밍(메시징) 플랫폼. 현재 가장 많이 사용되는 대표적인 메시지 큐(Message Queue)이다.  
※ **메시지 큐(Message Queue)** : 분산 시스템 간에 데이터를 주고받기 위한 시스템 및 프로그램  
※ 큐(Queue) : Stack 과 달리 FIFO

Apache Kakfa(이하 카프카)의 특징은 다음과 같다.
* Cluster 구성(카프카에서는 이를 '앙상블'이라 부른다)으로 분산처리가 가능
* **Publising - Subscription 구조** : Pub/Sub 구조
* Logback 기능 "이 부분 추가"

### Pub/Sub 구조
Pub/Sub 구조 내에서 Publisher 와 Consumer 는 서로의 존재에 대해 전혀 알지 못한다(그보다는 알 필요가 없다는 것이 정확할 듯).  
가운데에 놓인 Broker 를 두고, 브로커를 향해 메시지를 POST 하고, 브로커로부터 메시지를 PULL 해오는 것일 뿐.  
Broker 에 저장된 Topic 내의 Message 를 Publishing 하고 Subscripting 하는 구조이다.

이 Pub/Sub 구조를 사용하는 다른 서비스를 간략히 적어보자면,
#### AWS
* MSK : AWS 내의 Kafka 시스템으로, EC2 생성부터 설치, 클러스터화 및 보안 설정까지 AWS 내에서 지원해준다.
* SQC : 서버리스 서비스라는 장점에 반해 데이터 무결성 보장이 되지 않는 단점이 있다. 
#### GCP(Google Cloud Platform)
* Google Cloud Pub/Sub

## Kafka 구성
"그림 추가"  
카프카에서 중요한 용어가 바로 **Topic**과 **Message**이다.  
* Topic : 카프카의 데이터를 분류하는 단위로, Elasticsearch 의 인덱스와 비슷한 개념이라 할 수 있다.
* Message : 카프카로 보내지고, 카프카에 보관되고, 카프카에서 꺼내어져가는 데이터로,  Elasticsearch 의 Document, Data 와 비슷한 개념이라 할 수 있다.

카프카는 다음의 세 구성으로 이루어져 있다.
* **Producer(Publisher)** : 브로커를 향해 메시지를 저장하기 위해 보내는(POST) 주체
* **Broker** : 프로듀서로 인해 받은 메시지를 보관하고, 컨슈머로부터 메시지를 꺼내어져가는(GET) 주체.  
  기본적으로 브로커는 주체적으로 메시지를 받거나 보내는 행동을 취하지 않는다.  
  프로듀서가 프로듀싱하고, 컨슈머가 컨슘해 가는 것
* **Consumer(Subscriber)** : 브로커로부터 메시지를 직접 가져오는 주체  
  컨슈머가 자신의 역량에 맞게 메시지를 PULL 해온다.

카프카의 설치 및 기본 관리를 위해 필요한 프로그램은 아래와 같다.
* **Zookeeper** : 카프카 뿐 아니라 Hadoop 등의 Apache 솔루션에서 분산처리 관리를 위한 소프트웨어
* **Kafka Server** : Broker 역할
* **Kafka Manager** : Yahoo 사에서 개발한 서드파티 솔루션으로, 카프카 클러스터 및 각 서버, 토픽 등을 웹 화면에서 간결한 UI 로 관리할 수 있는 툴

## Install Kafka
카프카를 설치할 때 알아두면 좋은 내용을 간단하게 작성한다.

### 포트 번호
|Solution|Port Number|Remarks|
|------|---|---|
|Zookeepr|TCP/2181|-|
|Kafka|TCP/9092|-|
|Kafka Manager|TCP/9000(Web), TCP/9999(Node간)|-|
|JMX|38080|Manage 쪽에서 보는 포트번호 "이 부분 설명 추가"|

### Partition 관리
Kafka 역시 메시지를 보관할 때 Shard 단위(**Partition**)로 분산하여 저장하기 때문에 성능 및 안정성이 높다.  
메인이 되는 파티션을 **Leader**, 복사본을 **Replica 및 Follower** 라 하며, 기본적으로 파티션의 수는 브로커의 수와 맞춰준다.  
메시지의 Write 를 Leader 가 담당하며, Follower 는 Read 를 담당하는데, 항상 싱크를 맞추고 있다가 Leader 에 문제가 생길 경우 Follower 중 하나가 Leader 로 선출된다.

### Java Heap Memory
Kafka 는 Java 기반의 솔루션이므로 JVM 메모리 계산이 중요하다.  
일반적으로 Kafka : OS 의 힙 메모리 비율은 1 : 2 로. Kafka : Zookeeper 을 2 : 1 로 한다.  
예를 들어, 8G 메모리의 서버에 카프카와 주키퍼를 함께 올릴 때, 카프카 2G 주키퍼 1G 정도의 설정을 준다고 한다. "이유 설명 추가"

### Thread
컨슈머의 수는 브로커 서버의 CPU Therad 수와 같게 설정하는 것이 최적이지만, 이는 CPU 의 낭비가 심하기 때문에 보통 배수로 맞춰준다.

## Offset, Lag 관계
"그림 추가"  
퍼블리싱된 메시지는 설정에 맞게 지정된 토픽에 저장된다.  
각각의 토픽은 Partition 으로 분산되어 저장되고, 각 파티션에서 한 단위의 메시지가 저장되는 공간의 단위를 Log 라 부른다.  
즉, 메시지는 한 칸의 로그에 자동으로 append 되어 저장되는 방식인 것이다.

이 때, 로그에 저장된 메시지들의 위치를 나타내는 값이 바로 **Offset** 이다.  
파티션 내에 있는 레코드(로그)를 고유하게 식별하는 ID 라 할 수 있어, 마치 MongoDB 의 Shard ID 이자 Elasticsearch 의 _id 와 같다.  

한편, Offset 은 다음의 두 종류로 나뉜다.
* Log-End Offset : 어떤 토픽의 파티션에 저장된 데이터의 끝을 나타내는 ID. 즉, 프로듀서에 의해 커밋된(투입된) 마지막 메시지의 오프셋값
* Current Offset : 컨슈머 그룹에 의해 커밋된(가져가진) 마지막 메시지의 오프셋값

이 두 Offset 값이 왜 중요한가 하면, 두 오프셋값의 비교를 통해 컨슈머 그룹이 제대로 메시지를 컨슘해갔는지(Consumer 이 지연되고 있는지 아닌지) 판단할 수 있기 때문이다.
즉, (Log-End Offset - Current Offset) 값이, 프로듀싱된 메시지를 컨슈머 그룹이 제대로 가져가지 않은 값이며, 그 값을 **Lag** 이라 한다.

## Kafka 의 차별점(다른 메시지 큐와 비교)
카프카가 다른 메시지 큐와의 차별점을 두고 있는 부분은 아래와 같다.
* 앙상블이라는 Cluster 구조를 취하고 있어 **대용량 데이터를 다루는 데 특화**되어 있다.
* 브로커가 직접 컨슈머에 데이터를 Put 하는 방식이 아닌 **컨슈머가 자체적으로 Get 해가는 방식으로, 컨슈머 측에서 효율적인 조정**이 가능하다.
* 다만, 위의 구성으로 인해 RabbitMQ 등의 **다른 메시지 큐의 기능을 제공하지 않는다**는 한계점이 있다.

## 참고
* https://www.confluent.io/get-started/?product=software
* https://www.redhat.com/ko/topics/integration/what-is-apache-kafka
* https://zzsza.github.io/data/2018/07/24/apache-kafka-install/
* https://medium.com/@umanking/%EC%B9%B4%ED%94%84%EC%B9%B4%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0-%ED%95%98%EA%B8%B0%EC%A0%84%EC%97%90-%EB%A8%BC%EC%A0%80-data%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0%ED%95%B4%EB%B3%B4%EC%9E%90-d2e3ca2f3c2
