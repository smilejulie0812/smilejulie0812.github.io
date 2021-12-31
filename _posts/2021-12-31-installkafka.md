---
title: "Kafka 설치하기"
excerpt_separator: "<!--more-->"
categories:
  - Kafka
tags:
  - Kafka
  - Install
sidebar:
  nav: "docs"
---

Redis 초기 설치하는 방법을 정리하면서 의외로 OS 쪽 설정이나 conf 쪽 내용 복습도 되고 좀 더 내용을 찾아볼 수 있는 기회가 되어서, Kafka 도 같이 정리해 두기로 했다(나중에 Elastic Stack 쪽은 아예 시리즈물로 써볼까 싶기도 하고).

설치하고자 하는 Kafka 구성은 한 대의 서버에 Zookeeper 와 Kafka 를 함께 올리는 방식으로 한다.

## 사전 준비

### JDK 설치

Kafka 는 Java 기반의 솔루션이므로 JDK 설치가 필요하다.

```bash
sudo apt-get update
sudo apt-get install openjdk-<JDK버전>-jdk
java -version
```

## 설치

Kafka 를 설치하기 위한 바이너리 아카이브 파일은 아래 두 루트를 통해 다운로드받을 수 있다.

- Apache 사에서 다운로드 : 무료 오픈소스
    - [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)
- Confluent Community 다운로드 : 무료이나 오픈소스는 아님
    - [https://docs.confluent.io/platform/current/installation/installing_cp/zip-tar.html#get-the-software](https://docs.confluent.io/platform/current/installation/installing_cp/zip-tar.html#get-the-software)

여기에서는 Apache 에서 제공하는 바이너리 파일을 다운로드받아 설치하는 방법을 소개한다.

```bash
### 바이너리 파일 다운로드
wget https://archive.apache.org/dist/kafka/<버전>/kafka_2.12-<버전>.tgz
### 바이너리 파일 압축 풀기
tar -zxvf kafka_2.12-<버전>.tgz
### 심볼릭 링크 생성하여 차후 버전 관리 등에 대비하기
ln kafka_2.12-<버전> kafka
```

## 환경파일 설정

### 전용 유저 및 그룹 설정

Kafka 및 Zookeeper 를 실행할 전용 유저와 그룹을 생성한다.

```bash
sudo groupadd <그룹명>
### 로그인 기능을 필요하지 않으므로 nologin 으로 지정
sudo useradd -s /sbin/nologin -M -g <그룹명> <유저명>
```

### 데이터 보관용 디렉토리 생성

Kafka 및 Zookeeper 에서 보관하는 데이터(Kafka 에서는 이를 log 라고 표현한다 ~~진짜 로그 파일과는 다르다~~)용 디렉토리를 미리 생성한 후 config 파일에 설정한다.

```bash
### 아래 디렉토리명은 예시
### Zookeeper 용 데이터 디렉토리 생성
mkdir /var/lib/zookeeper
### Kafka 용 로그(데이터) 보관 디렉토리 생성
mkdir /var/lib/kafka
```

### conf 설정

config 설정은 일부만 포함되어 있다

- <path>/config/server.properties

```bash
broker.id=<브로커 고유 아이디>

### zookeeper 접속 IP 리스트
zookeeper.connect=<Zookeeper #01 IP 주소>:2181,<Zookeeper #02 IP 주소>:2181,<Zookeeper #03 IP 주소>:2181

listeners=PLAINTEXT://<Broker 가 내부적으로 바인딩하는 IP 주소>:9092
advertised.listeners=PLAINTEXT://<Producer 및 Consumer 에게 보일 IP 주소>:9092

### 로그(데이터) 보관 주기(hour)
log.retention.hours=168

### 로그(데이터) 파일 보관 디렉토리 경로
log.dirs=/var/lib/kafka
### 로그(데이터) 파일 1개의 최대 크기(기본값 1GB)
log.segment.bytes=1073741824

### 토픽의 메시지를 디스크로 플러시하기 전까지 메모리에 보관할 최대 시간(ms)
log.flush.interval.ms=5000
### 토픽의 메시지를 디스크로 플러시하기 전까지 메모리에 보관할 메시지 수
log.flush.interval.messages=20000

### 클러스터 내의 코디네이터가 리밸런싱을 수행하기 전에, 다른 컨슈머 
group.initial.rebalance.delay.ms=10000

## 오프셋 토픽의 replication factor 수 설정. 클러스터 크기가 이 값을 충족하지 않으면 내부 토픽 생성에 실패함
offsets.topic.replication.factor=3
## 트랜잭션 토픽의 replication factor 수 설정. 클러스터 크기가 이 값을 충족하지 않으면 내부 토픽 생성에 실패함
transaction.state.log.replication.factor=3

### 토픽 당 기본 파티션 수
num.partitions=1 

### topic 삭제 허가 설정
delete.topic.enable=true
### 자동 topic 생성 허가 설정
auto.create.topics.enable=true
```

- <path>/config/zookeeper.properties

```bash
### Zookeeper 데이터 보관 디렉토리 경로
dataDir=/var/lib/zookeeper

### 클라이언트가 접속할 Zookeeper 포트번호로 기본값은 2181
clientPort=2181
### 클라이언트에서 동시 접속하는 개수 제한으로 기본값 60, 무제한 설정시 0
maxClientCnxns=0

### Follower가 Leader와의 연결 시도시의 tick 제한 횟수
initLimit=5                             
### Follower가 Leader와 연결된 후 동기화되기 위한 tick 제한 횟수
syncLimit=2                             

## Zookeeper 앙상블을 이룰 서버 설정
server.1=<IP 주소>:2888:3888
server.2=<IP 주소>:2888:3888
...
server.N=<IP 주소>:2888:3888
```

### Zookeeper ID 파일 설정

- <dataDir>/myid
    
```bash
# <path>/config/server.properties 의 broker.id 값과 동일한 값을 저장한다
```

### Heap Size 조정

- <path>/bin/kafka-server-start.sh

```bash
### Kafka 와 OS 메모리의 비율을 1:2 정도로 잡는 게 일반적
### Kafka 자체가 Heap 공간을 주의해서 사용하므로 5GB 이상은 설정하지 않아도 된다
export KAFKA_HEAP_OPTS="-Xmx2G -Xms2G"
```

- <path>/bin/zookeeper-server-start.sh

```bash
### Zookeeper 와 Kafka 의 Heap Size 비율을 1:2 정도로 잡는 게 일반적
### 다만 모니터링을 위해 1GB 정도를 잡아준다
export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
```

## systemd 설정

kafka 는 zookeeper 가 먼저 시작된 후에 시작해야 하기 때문에, 이 점을 유념하여 systemd 파일을 작성한다.

### zookeeper systemd 파일 작성

- /usr/lib/systemd/system/zookeeper.service
    
    ```bash
    [Unit]
    Description=Zookeeper server
    Requires=network.target remote-fs.target
    After=network.target remote-fs.target
     
    [Service]
    Type=simple
    Environment=JAVA_HOME=/usr/lib/jvm/java-<버전>-openjdk-amd64
    SyslogIdentifier=zookeeper
    WorkingDirectory=<path>
    RestartSec=0s
    ExecStart=<path>/bin/zookeeper-server-start.sh <path>/config/zookeeper.properties
    ExecStop=<path>/bin/zookeeper-server-stop.sh
     
    [Install]
    WantedBy=multi-user.target
    ```
    

### kafka systemd 파일 설정

- /usr/lib/systemd/system/kafka.service
    
    ```bash
    [Unit]
    Description=Kafka server
    Requires=zookeeper.service
    After=zookeeper.service
     
    [Service]
    Type=simple
    Environment=JAVA_HOME=/usr/lib/jvm/java-<버전>-openjdk-amd64
    SyslogIdentifier=kafka
    WorkingDirectory=<path>
    RestartSec=0s
    ExecStart=<path>/bin/kafka-server-start.sh <path>/config/server.properties
    ExecStop=<path>/bin/kafka-server-stop.sh
     
    [Install]
    WantedBy=multi-user.target
    ```
    

### /etc/systemd 경로로 링크

- /usr/lib/systemd : 패키지 다운로드로 설치했을 때 system 파일이 저장되는 장소
- /etc/systemd : 시스템 관리자가 관리하는 영역

보통의 경우, /usr/lib/systemd 쪽에 저장된 시스템 파일에 대해 /etc/systemd 쪽에 유닛을 추가하면 오버라이드되지만, 이 경우 /usr/lib/systemd 부터 직접 생성했으므로 soft copy 로 링크를 걸어둔다.

```bash
cd /etc/systemd/system
ln -s /lib/systemd/system/zookeeper.service zookeeper.service
ln -s /lib/systemd/system/kafka.service kafka.service
```

## 참고

- [https://godekdls.github.io/Apache Kafka/contents/](https://godekdls.github.io/Apache%20Kafka/contents/)
- [https://velog.io/@jwpark06/Kafka-시스템-구조-알아보기](https://velog.io/@jwpark06/Kafka-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B5%AC%EC%A1%B0-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
- [https://twowinsh87.github.io/etc/2018/08/03/etc-kafka-3/](https://twowinsh87.github.io/etc/2018/08/03/etc-kafka-3/)
