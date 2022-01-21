---
title: “Redis Sentinel & Cluster”
excerpt_separator: "<!--more-->"
categories:
  - Redis
tags:
  - Redis Sentinel
  - Redis Cluster
sidebar:
  nav: "docs"
---
레디스는 업무상 자주 다루지 못하는 솔루션이라 그런지 아직 많은 부분을 모르는 게 아쉽다.

레디스도 버전 4 부터 Cluster 구성이 가능해지면서 거의 클러스터화 되었다고 생각하지만, 이전에 많이 쓰인 Sentinel 과의 차이점을 간단하게 정리해 보면 어떨까 해서 작성해보는 포스트.

## Redis Sentinel

{% raw %}<img src="https://smilejulie0812.github.io/assets/images/redisSentinelAndCluster-1.png" alt="">{% endraw %}

- Redis 를 **Master / Slave** 로 구성해서 **HA 를 가능케** 함
- Master / Slave 의 **관리 역할** (**Sentinel**) 이 별도로 존재

- Master 에 장애가 발생했을 때, Active / Standby 역할을 바꾸어 Slave 가 Master 의 역할을 대신함
    - Slave 가 Master 에 선정되는 기준 : **Replica-priority 값이 가장 작은 Slave 가 Master 로 선정**됨

- Master / Slave 는 항상 동기화하여 데이터 정합성을 유지
    - Master / Slave 간에는 **Redis Pub/Sub 기능을 이용해서 동기화**됨
    - Redis Sentinel 구성은 **비동기 복제**를 사용하기 때문에 완벽한 데이터의 정합성을 기대하기 어렵다
    
- 단일 Master 구조로, 데이터 사이즈가 커지게 되었을 때 **확장을 위해서는 Scale-Up** 을 해야 함

- 안정적인 운영을 위한 구성 설계
    - 3 개 이상의 Sentinel 을 만들되, 각각의 노드는 물리적인 영향을 받지 않는 서버 혹은 인스턴스에 설치
    - Sentinel 은 Redis 노드와 같은 서버에 구성해도, 별도 서버에 구성해도 상관없음

## Redis Cluster

{% raw %}<img src="https://smilejulie0812.github.io/assets/images/redisSentinelAndCluster-2.png" alt="">{% endraw %}

- Sentinel 보다 발전된 형태
- **샤딩 기능**을 통해 **데이터를 분산 저장**할 수 있음
- CRC16 알고리즘을 이용해서 각 Document 의 Key 가 해시 함수를 통해 특정 값으로 변환되고(**해시 슬롯**),
그 해시 슬롯 값에 맞게 Cluster 의 Master 노드에 할당 및 적재
- 각각의 Redis 노드는 **Gossip Protocol 을 통해 노드들 간 직접 통신 가능**
- Master 노드 장애 발생 시 Slave 중에서 하나가 Master 로 선출되어 **무중단 서비스를 가능**케 함

- Master / Slave 를 복수로 구성할 수 있음
- 복수의 master 구조가 가능해서 데이터 사이즈가 커지게 되면 **Scale-Out 으로 확장 가능**

### 참고

- [http://redisgate.kr/redis/sentinel/sentinel.php](http://redisgate.kr/redis/sentinel/sentinel.php)
- [http://redisgate.kr/redis/cluster/cluster.php](http://redisgate.kr/redis/cluster/cluster.php)
