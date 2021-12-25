---
title: "Redis をインストールしてみた"
excerpt_separator: "<!--more-->"
categories:
  - Redis
tags:
  - Redis
  - Japanese
sidebar:
  nav: "docs"
---
redis 초기 인스톨 정보를 정리한 후 일본어로 번역하는 작업

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

[https://experiences.tistory.com/18](https://experiences.tistory.com/18) ← 이 내용 이해해서 설정

```bash
### Socket Max Connection 값 설정 변경
sudo sysctl -w net.core.somaxconn=1024
### 서버 재시작 후에도 설정 유지
sudo echo "net.core.somaxconn=1024" >> /etc/sysctl.conf
### 설정 확인
sudo sysctl -a | grep somaxconn
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
### 워크 디렉토리 작성

```bash

```

### conf 설정

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
