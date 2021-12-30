---
title: "Redis Cluster 설치하기"
excerpt_separator: "<!--more-->"
categories:
  - Redis
tags:
  - Redis
  - Install
sidebar:
  nav: "docs"
---
Redis Cluster 구성을 위한 설치 방법을 정리해서 기재한다.

## 버전 정보

- OS : Ubuntu 20.04
- Redis Version 6.2

## 사전 준비

### gcc-c++ 설치

Redis 는 C 언어로 쓰여진 솔루션이므로 설치할 때 컴파일러가 필요하다. 미리 gcc-c++ 를 설치해둔다.

```bash
sudo apt-get install gcc-c++
```

### 메모리 설정

Redis 와 같이 메모리를 많이 점유하는 서비스의 경우, 메모리가 부족해지면 OOM-Killer 가 동작해서 OutOfMemory 이슈로 kill 되어버릴 수 있다.

이를 방지해서 메모리가 부족해져도 Swapping 등으로 서비스가 죽지 않도록 항상 메모리의 overcommit 을 허용해 주는 설정을 적용할 필요가 있다.

```bash
### 커널 설정 변경 : 항상 overcommit 허용
### vm.overcommit_memory=0 : 디폴트값. 서버 상태에 따라 자주적으로 조정하도록(Heuristic) 설정
### vm.overcommit_memory=2 : 제한적 overcommit 허용
sudo sysctl vm.overcommit_memory=1
### 서버 재시작 후에도 설정 유지
sudo echo "vm.overcommit_memory=1" >> /etc/sysctl.conf
### 설정 확인
sudo sysctl -a | grep vm.overcommit_memory
```

### TCP Backlog 설정
TCP 3-way handshake 과정에서 클라이언트 -> 서버 로 보낸 응답이 대기하는 장소가 Backlog Queue 인데, 이 값을 조정해줘야 응답 초과로 퍼포먼스 저하를 방지할 수 있다.
* net.core.somaxconn : ESTABLISHED 상태의 소켓을 위한 큐
* net.ipv4.tcp_max_syn_backlog : SYN_RECEIVED 상태의 소켓을 위한 큐

```bash
### Socket Max Connection 값 설정 변경
sudo sysctl -w net.core.somaxconn=1024
### Syn Backlog 값 설정 변경
sysctl -w net.ipv4.tcp_max_syn_backlog=1024
### 서버 재시작 후에도 설정 유지
sudo echo "net.core.somaxconn=1024" >> /etc/sysctl.conf
sudo echo "net.ipv4.tcp_max_syn_backlog=1024" >> /etc/sysctl.conf
### 설정 확인
sudo sysctl -a | grep somaxconn
sudo sysctl -a | grep syn_backlog
```

### THP Disabled 설정

Linux 는 메모리를 ‘page’ 라는 단위로 관리하며, 페이지의 크기는 기본값 4K 로 고정되어 있다.

다만 메모리 크기가 늘어남에 따라 고정된 페이지의 수가 점점 늘어나게 되고, 그 결과 메모리에 access 하는 과정에서 수많은 페이지로 인해 overhead 가 발생하는 문제가 생기게 되어, 해결 방법으로 페이지 크기를 확대하는 Huge Page 개념이 생겨나게 되었다.

이 Huge Page 을 자동으로 관리해 주는 기능이 바로 THP(Transparent Huge Pages) 이다.

다만 이 THP 기능을 사용하는 과정에서 퍼포먼스 저하 이슈가 발생하게 되었고(~~페이지를 늘렸다 줄였다 하는데 리소스를 사용해 버릴테니...~~), 퍼포먼스를 중시하는 서비스를 사용할 때에는 THP 기능을 비활성화하도록 권장하고 있다.

```bash
### THP 비활성화 설정
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
### 서버 재시작 후에도 설정 유지
vi /etc/rc.local
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

### Max Open Files 설정
리눅스는 실제 파일은 물론 네트워크 및 소켓에 대한 접속의 단위가 **파일**이다(File System).

OS 전체 혹은 특정 사용자에 대해 기본적으로 file 을 열 수 있는 값이 한정되어 있는데, Redis 가 대규모로 사용되는 과정에서 클라이언트의 수가 많아지거나 레디스 서버가 늘어나게 되면 아래의 이슈가 발생할 가능성이 생긴다.

- redis.conf 의 설정(maxclients 값)과 OS 상의 max open files 설정값의 불일치 및 기준치 초과 이슈
- too many open files 이슈

위의 이슈를 미리 방지하기 위해 두 가지 설정을 추가한다.

1. redis.conf 파일의 maxclients 값 조정 : 아래에 redis 설치 후 설정 파일 적용할 때 추가한다
2. OS 상에서 max open files 설정값 조정

```bash
### 현재 max open files 값 확인
ulimit -a
ulimit -Hn # Hard Limit 값
ulimit -Sn # Soft Limit 값
### max open files 설정
ulimit -n <max open files>
### 서버 재시작 후에도 설정 유지
vi /etc/security/limits.conf
# redis 사용자에 대해 Soft/Hard Limit 값 설정
redis soft nofile <max open files>
redis hard nofile <max open files>
* hard nofile <max open files> # * 로 설정하면 모든 사용자에 대해 Limit 값 설정
```

## 설치

### 다운로드

```bash
### 설치 전 apt-get 업데이트
sudo apt-get update
### 인스톨 버전 확인해서 다운로드
wget https://download.redis.io/releases/redis-<버전>.tar.gz
```

### 컴파일 및 설치

```bash
### 다운로드 파일 압축 풀기
tar -zxvf redis-<버전>.tar.gz
cd redis-<버전>.tar.gz
### 컴파일 및 설치
sudo make
sudo make install
```

## 환경파일 설정

### 전용 유저 및 그룹 설정

Redis 를 실행할 전용 유저와 그룹을 생성한다.

```bash
sudo groupadd <그룹명>
### 로그인 기능을 필요하지 않으므로 nologin 으로 지정
sudo useradd -s /sbin/nologin -M -g <그룹명> <유저명>
```
### 작업 디렉토리 작성

Redis 프로세스를 관리할 작업용 디렉토리를 생성한다(conf 파일이나 관련 로그 디렉토리 관리 등을 이 곳에서 할 수 있다).  
※ 물론, conf 파일이나 실행/중지 스크립트 전용 디렉토리는 /etc/redis 에, 로그 전용 디렉토리를 /var/log/redis 에 보관하도록 설정하여  
다른 debian package 솔루션과 경로를 통일하는 것도 한 가지 방법이다(여기에서는 전용 디렉토리에서 한 번에 관리하는 경로로 설정한다).

```bash
### redis 전용 디렉토리 생성
mkdir <관리할 디렉토리 경로>/redis
### redis 로그 디렉토리 생성
mkdir <관리할 디렉토리 경로>/redis/logs
### 디렉토리 권한 설정
chmod -R 755 <관리할 디렉토리 경로>/redis
chown -R redis:redis <관리할 디렉토리 경로>/redis
```

### conf 설정
* redis/redis.conf

```bash
### 로컬 환경 뿐 아니라 모든 IP 와의 연결을 허용
bind 0.0.0.0

### RDB(Redis 의 현재 메모리에 대한 dump 생성 기능) 설정 중 save second changes 설정
#save 900 1   ### 900초 안에 1번 이상 데이터 변경이 생길 때 RDB 생성
#save 300 10    ### 300초 안에 10번 이상 데이터 변경이 생길 때 RDB 생성
#save 60 10000      ### 60초 안에 10000번 이상 데이터 변경이 생길 때 RDB 생성
save ""     ### Redis 를 캐시로서 사용하므로, RDB 생성하지 않음

### systemd init 시스템을 통한 Redis 관리 허용
supervised systemd

### 클러스터 지원을 활성화
cluster-enabled yes

### 전체 메모리 중 Redis 가 최대로 사용할 크기 설정. 서버/인스턴스의 메모리 사이즈에 따라 달라진다
maxmemory 1g

### maxmemory 를 초과할 때 데이터 삭제 방식 설정
#maxmemory-policy allkeys-lfu     ### LFU 방식으로 데이터 삭제. 캐시미스 발생시 가장 적은 빈도로 사용된 캐시를 삭제하여 공간확보
#maxmemory-policy allkey-random   ### 랜덤 방식으로 데이터 삭제
maxmemory-policy allkeys-lru  ### LRU 방식으로 데이터 삭제. 캐시미스 발생시 가장 마지막에 로드된 캐시를 삭제하여 공간 확보

### 패스워드 설정
requirepass <해시 암호>
masterauth <해시 암호>

### 마스터 유저명 설정 무효화 : 버전 6 Redis Cluster 구성 시에는 필히 해당 옵션을 무효화(주석처리)
#masteruser master

# 클러스터 구성 내용을 저장하는 파일명을 지정
cluster-config-file <filename.conf>

# 클러스터 노드가 실패한 것으로 간주되지 않고 사용할 수 없는 최대시간
# 지정된 시간 이상 Master 노드에 도달 할 수 없는 경우 Slave 노드로 failover 됨
cluster-node-timeout <milliseconds>

# 클러스터 일부 노드가 다운되어도 클러스터 전체가 다운되지 않고 운영할 수 있는 여부를 선택
cluster-require-full-coverage  <yes/no>

# yes 이고 bind 가 지정되어 있으면 지정한 IP 로만 접속 가능
# yes 이고 bind 가 지정되어 있지 않으면 127.0.0.1(로컬 환경)로만 접속 가능
protected-mode <yes/no>

# 로그 출력 설정
dir <경로>/redis/log/
logfile redis.log

# max open file 설정 : 기본값 10000
# OS 기본값은 1024 이므로, OS 에 맞추기 위해서는 이 maxclients 값을 992 로 설정
# 아니면 ulimit 명령어로 OS 기본값을 올리는 것도 방법(위에서 올리는 방법 소개)
maxclients <max open file 값>
```

### 서비스 시작 및 종료 스크립트 작성
Redis 는 서비스 시작 및 종료를 위한 명령어 없이 프로세스 단위로 다루기 때문에, 시작과 종료를 위한 간단한 스크립트를 작성해 두면 편리하다.  
#### 서비스 시작 스크립트
* start-redis.conf
```bash
#!/bin/sh -
/redis/redis-6.0.9/src/redis-server /redis/redis-6.0.9/redis.conf
```
#### 서비스 종료 스크립트
* stop-redis.conf
```bash
#!/bin/sh -
/redis/redis-6.0.9/src/redis-cli -p 6379 shutdown
```

## systemd 설정
### systemd 파일 작성 및 추가
systemd 파일을 작성하여 서비스 관리를 편하게 할 수 있다.
* /usr/lib/systemd/system/redis.service  

```bash
[Unit]
Description=Redis-Server
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/redis/redis-start.sh
ExecStop=/redis/redis-stop.sh

[Install]
WantedBy=multi-user.target
```
### 서비스 enabled 설정
설정한 서비스를 enabled 로 하여 서버 재시작 후에도 자동으로 서비스를 시작할 수 있도록 한다.
```bash 
systemctl enable redis.service
```

## 참고
- [http://redisgate.jp/redis/configuration/redis_start.php](http://redisgate.jp/redis/configuration/redis_start.php)
- [https://qiita.com/KurosawaTsuyoshi/items/f8719bf7c3a10d22a921](https://qiita.com/KurosawaTsuyoshi/items/f8719bf7c3a10d22a921)
- [https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04-ja](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04-ja)
- [https://www.psjco.com/26](https://www.psjco.com/26)
- [https://experiences.tistory.com/18](https://experiences.tistory.com/18)
