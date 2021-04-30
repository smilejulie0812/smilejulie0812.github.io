---
title: "대용량 트래픽 처리 시스템과 Swap 관계에 대한 고찰"
excerpt_separator: "<!--more-->"
categories:
  - Linux
tags:
  - linux
  - swap
  - elasticsearch
sidebar:
  nav: "docs"

---
## 서론
화려하게 실패한 첫 면접에 대한 반성으로 정리해보는 swap memory에 대한 내용 정리를 해 보고자 한다. 이 포스트는 자칭 ELK 애호가인 주제에 기본 중의 기본에 제대로 답하지 못한 반성으로부터 시작한다.

**"대용량 트래픽 처리 시스템에서 Swap을 무효화하는 추세인데, 그 이유는 무엇인가요?"**

## Swap의 정의를 묻는다면
평화롭게 움직이는 서버라면 실행되는 프로세스가 기본적으로 물리 메모리(RAM)에 할당되어 실행될 것이다. 하지만 프로세스가 과다하게 실행되어 할당될 메모리 공간이 부족하다면 어떻게 될까?  
이 때 필요한 공간이 바로 Swap(스왑) 메모리이다. **물리 메모리를 모두 소모하여 공간이 부족해진 경우, 하드 디스크(혹은 보조 기억장치)의 일부분을 마치 메모리와 같이 사용하게 되는데, 이 공간**을 바로 스왑 메모리라 한다.
즉, 스왑이란 메모리 공간의 부족 문제를 해결하기 위해 일정 부분으로 할당된 하드디스크의 공간을 이용하여 여분의 메모리 공간을 확보하는 기술인 것이다.

이 때, (새 프로세스 실행에 의한) 메모리 부족으로 인해 실행되고 있는 프로세스를 하드디스크로 보내는 작업을 **swap-out**, 프로세스를 다시 RAM으로 불러들이는 작업을 **swap-in**이라 한다.

## Swap 공간을 확보하자
스왑 공간을 확보하기 위한 방법은 크게 두 가지로 나뉜다. **스왑 파티션**을 생성하거나, **스왑 파일** 형태로 만들거나. 이 포스트에서는 스왑 파일을 생성하는 방법을 정리해보도록 한다.

#### dd 명령어로 스왑 파일을 생성하기
\# dd if=/dev/zero of=/swapfile bs=1M count=1024
{: .notice--primary}

#### 스왑 파일의 허가권을 설정하기
\# chmod 600 /swapfile
{: .notice--primary}
스왑 파일은 로트 사용자만 읽고 쓰기가 가능하도록 허가권을 설정해준다.

#### mkswap 명령어로 스왑 파일 초기화하기
\# mkswap /swapfile
{: .notice--primary}

#### swapon 명령어로 스왑 파일을 시스템에 인식시키기
\# swapon /swapfile
{: .notice--primary}

#### 스왑 상태 확인하기
\# swapon -s
{: .notice--primary}

#### 서버를 재기동할 때 자동으로 swap을 유효화하기
\# vi /etc/fstab  
/swapfile swap swap defaults 0 0
{: .notice--primary}

## 기타 Swap 관련 명령어

#### free 명령어
물리 메모리나 스왑 메모리 등, 메모리의 상태를 출력하기 위해 사용하는 명령어가 **free 명령어**이다.

#### 스왑 파일을 해제할 때
\# swapoff -v /swapfile
{: .notice--primary}

#### 스왑 파일을 삭제할 때
\# rm /swapfile
{: .notice--primary}

## Elasticsearch에 Swap 설정은 금물?!
위의 내용을 생각하면 스왑 공간은 여러모로 중요한 역할을 하는 것 같은데 왜 대용량 트래픽을 처리할 때 스왑 메모리를 무효화시키는 것일까? 그 이유는 바로 디스크에 있다.
대용량 트래픽 시스템에서는 말 그대로 굉장한 양의 프로세스를 처리하기 때문에, RAM이 꽉 찰 경우가 빈번히 일어날 것이다. 이 때 스왑 사용이 유효화되어있을 경우, 당연히 **많은 프로세스가 스왑 공간을 사용하기 위해 하드디스크로 내려올 것이고, 그 결과 하드디스크의 성능이 떨어져 시스템 전체의 성능에 막대한 영향을 끼치게 될 것**이다.

Elasticsearch야말로 메모리 상에서 많은 양의 데이터를 다루며 인덱싱 및 검색 등의 기능을 사용하므로, Elasticsearch를 실행할 때 스왑 기능이 살아있다면 치명적인 성능 저하가 야기될 수 있을 것이다(100마이크로초로 진행되는 작업이 순식간에 10밀리초로 작업하게 된다고 한다(...)).
이를 방지하기 위해, Elasticsearch에서는 다음의 세 가지 방법으로 스와핑을 중지하도록 추천한다.

#### 1. sysctl 명령어를 사용하여 스와핑 무효화
실제 프로젝트에서 사용한 방법이다. elasticsearch 전용 서버이므로 평상시에는 스와핑을 제어하지만, 응급 메모리 상황에서는 OS가 스와핑 기능을 사용할 수 있도록 설정하였다.
\# sysctl -w vm.swappiness = 1  
{: .notice--primary}
OR  
sudo vi /etc/sysctl.conf  
vm.swappiness=1  
sysctl -p
\# 
{: .notice--primary}
참고) vm.swappiness = 0 으로 설정하면 일부 커널에서 되려 OOM-killer를 불러와서 elasticsearch 프로세스를 kill 해버릴 수 있다고 한다(...)

#### 2. 영구적인 스와핑 무효화
\# swapoff - a  
/etc/fstab 에서 영구적으로 스와핑 무효화 설정
{: .notice--primary}

#### 3. elasticsearch.yml 내에서 부분적으로 스와핑 무효화
내 경우에는 서버 관리자 권한을 가지고 있었기 때문에 OS 측에서 설정이 가능했지만, 혹시 root 유저 사용이 불가능하거나 sudo 명령어를 사용할 수 없는 경우, `elasticsearch.yml` 내의 설정을 통해 elasticsearch 움직임에 한해서 스와핑 기능을 무효화할 수 있다.
즉, JVM 자체에 설정하여, elasticsearch가 기동할 때 할당된 메모리를 lock한 상태로 기동해서 스와핑 기능을 사용하지 못하도록 설정하는 것이다.
bootstrap.mlockall: true # 버전 2 이상  
bootstrap.memory_lock: true # 버전 5 이상
{: .notice--primary}

Elasticsearch에 대해서는 웬만한 엔지니어보다 잘 알고 있다고 자부했지만 사실은 속빈 강정이었다는 사실을 깨닫게 되었다. 하지만 역시 이렇게 공부하는 건 너무나도 재밌어... 앞으로 기술 블로그를 통해, 내 4년 간의 커리어를 확실히 채워나가도록 노력해야지.

#### 참고 페이지
* [Swapping Is the Death of Performance](https://www.elastic.co/guide/en/elasticsearch/guide/current/heap-sizing.html#_swapping_is_the_death_of_performance)
* [ElasticSearch Heap 메모리 설정](https://www.nakjunizm.com/2017/09/07/ElasticSearch_Heap_Memory/)
* [Swapping 스와핑(Swap 스왑)이란?](https://jhnyang.tistory.com/103)
* [スワップ領域の作成](https://qiita.com/azusanakano/items/96e7c490c285d9ec1b1a)
* [SWAP領域](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-swapspace)
