---
title: "Redis 오리엔테이션"
excerpt_separator: "<!--more-->"
categories:
  - Kafka
tags:
  - Redis
  - NoSQL
sidebar:
  nav: "docs"
---
Redis 는 부담당자여서 그런가 아직 많은 내용을 다루지는 못했지만, 덕분에 **캐시**의 개념을 다시 한 번 살펴볼 기회가 된 것 같다.

지난 번 신입사원 입사용 Kafka 오리엔테이션 내용에 이어서 이번에는 Redis 오리엔테이션 내용을 정리해보고자 한다.

## Redis

- 메모리 기반(in-memory)의 Key-Value 형 NoSQL(비관계형 데이터베이스)
- **in-memory** : 데이터를 저장하는 장소가 평소 디스크가 아닌 메모리
    - 메모리를 저장소로 사용하는만큼 속도가 매우 빠르지만, 프로세스가 kill 되면 데이터도 소실
    - **Cache** 용도 → RDBMS 의 보조 역할로 사용(물론 DB 용도로 사용 또한 가능)

### 특징

- Redis Server : 1개의 **싱글 스레드**로 수행( 서버 1개에 여러 Redis Server 를 올릴 수 있다 )
    - 싱글 스레드이기 때문에 하나의 작업이 끝날 때까지 다른 작업이 대기하고 있어야 함
    - 버전 6.x 부터 멀티 스레드로 사용 가능( 데이터 정합성 문제로 인해 cluster 구조 제외 )
    - <여기 싱글스레드 관련 그림 추가>
- Master - Slave 구조의 **클러스터** 구성 가능
    - <여기 클러스터 관련 그림 추가>
    - 처음 구축할 때 Master / Slave 의 대수 결정
        - Master Node : Redis 의 두뇌 역할 / 모든 명령어(Read & Write) 수행
        - Slave Node : Master 의 복제본으로, Read Only 수행(하나 보통은 데이터 정합성 문제로 Read 도 Master 가 수행)
    - 운영 도중 Master Node 가 셧다운되면, 같은 해시값의 데이터를 가지고 있는 Slave Node 중하나를 Master 로 자동 선출

### 사용 방법

#### Cache 로 Redis 사용하기

- 어떤 애플리케이션에서 데이터 요청이 들어왔을 때, RDBMS 에서 한 번 읽어 온 데이터를 Redis 에 보관해 두고, 데이터가 남아있는 동안 다시 같은 데이터 요청이 들어오면 RDBMS 를 거치지 않고 Redis 에서 바로 읽어가는 방식
- 물론 Redis 의 보관 설정 시간이 초과되면 메모리에서 해당 데이터가 삭제되므로, 그 이후 요청이 들어오면 다시 RDBMS 에서 읽어와 메모리로 임시보관시켜 사용한다

#### DB 로 Redis 사용하기

- 데이터를 처음부터 메모리에 올려놓고 DB 처럼 사용하는 방식
- 메모리에 보관되는 데이터는 RDB &  AOF 방식을 통해 백업
    - **RDB** 백업 방식 : 특정 시점의 메모리에 있는 데이터 전체를 바이너리 파일로 저장. AOF 방식 파일보다 작은 크기이므로 불러오는 데 상대적으로 시간이 적게 걸린다 → save 옵션
    - **AOF** 백업 방식 : 명령어가 실행될 때마다 해당 명령어 파일에 기록(기록 파일 기본값: appendonly.aof)하는 방법으로, 텍스트 파일로 저장 → appendonly 옵션
    - RDB 백업을 하루 n회, AOF 백업을 realtime 으로 진행함

### AWS Redis Service

- **ElastiCache** : AWS 내에서 Redis 서비스 사용( + Memcached )
- **MemoryDB** : AWS 자체에서 Redis 기능을 하는 서비스

### 기타 기능

- 버전 6.x 부터 ACL(유저/패스워드 관리) 기능이 추가되었으나, Standalone 구성에서만 가능하며 Cluster 구성에서는 아직 불가능 → Cluster 에서는 기존 masterauth 기능 사용(1개의 인증)

### 참고

- [https://it-jinsu.tistory.com/4](https://it-jinsu.tistory.com/4)
- [https://mozi.tistory.com/371](https://mozi.tistory.com/371)
