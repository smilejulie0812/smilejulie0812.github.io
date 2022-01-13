---
title: "vm.max_map_count 이해하기"
excerpt_separator: "<!--more-->"
categories:
  - Infra
tags:
  - vm.max_map_count
  - Elasticsearch
sidebar:
  nav: "docs"
---
이 설정은 메모리를 대량으로 사용하는 대용량 데이터 처리 시스템에서 굉장히 중요한 설정이다. 실제로 elasticsearch 든 kafka 든 redis 든 이 설정을 조정하는 경우가 대다수이기 때문에, 다시 한 번 내 언어로 정리해 두도록 한다.

## mmap : 메모리 맵 파일

데이터는 보통 파일 형태로 디스크에 저장된다. 하지만 모두들 알다시피 디스크에서 일일이 읽고 쓰는 속도에는 한계가 있다. 그럼 다들 생각할 것이다. 데이터 파일이, 디스크가 아닌 메모리에 있다면 읽고 쓰는, 즉 접근하는 속도가 훨씬 빨라지지 않을까?

여기서 힘차게 등장하는 개념이 디스크와 메모리 사이의 **페이지(page)**라는 개념이다. 어떤 프로세스가 움직일 때에는 주기억장치인 메모리에서 움직이는데, 자주 사용되지 않는 부분은 보조기억장치인 디스크로 내려서(옮겨서) 메모리의 여유 공간을 관리하는 행위를 우리는 **페이징(paging)**이라 하고, **메모리에서 디스크로 내릴 때, 한 번에 옮기는 크기의 단위**를 페이지라고 한다.

아니, 페이지는 알았는데 메모리 맵 파일은 뭐냐고?

메모리 맵 파일이란, 어떤 **프로세스가 할당된 가상 메모리 공간과 디스크에 있는 파일을 매핑(연결)**해서, **디스크가 아닌 가상 메모리 주소에 직접 접근하면서 데이터를 읽고 쓸 수 있는 기법**이다. 

이렇게 쓰면 좋은 점이 첫째, 데이터를 사용하는데 디스크까지 내려가지 않고 가상 메모리 상에서만 읽고 쓸 수 있으니 스피드가 빨라지는 것은 당연할 것이다. 

게다가 둘째로, 가상 메모리 공간에서 일정 사이즈로만 할당된 페이지 내에서 읽고 쓸 수 있기 때문에, 메모리 사용을 절약하며 효과적으로 사용할 수도 있을 것이다. 그럼 **아무리 큰 데이터를 처리하더라도, 필요한 데이터만 디스크에서 메모리에 위치한 매핑된 페이지로 불러와 사용할 수 있을 것**이다. 즉 이것이, **메모리 맵 파일이 대용량 데이터 처리를 할 때 매우 효과적인 이유**인 것이다.

## vm.max_map_count

그래, 메모리 맵 파일이 대용량 데이터 처리를 할 때 필요한 개념이라는 것까지 이해했다. 그럼 저 vm.max_map_count 는 메모리 맵 파일과 무슨 관계인데?

vm.max_map_count 는 Linux 의 sysctl 설정 변수이다. Linux 에서는 하나의 프로그램이 실행되면서 한 번에 너무 많은 파일에 접근하려고 하거나, 혼자서 너무 많은 메모리를 매핑해서 사용하는 것을 방지할 수 있는 옵션이 있다(~~제 아무리 큰 프로그램이라 한들 귀하신 메모리님을 혼자서 독차지하게 둘 수는 없지!~~). 그 옵션이 바로 **vm.max_map_count** 인 것이다. 즉, **하나의 프로그램 당 사용할 수 있는 메모리 맵 파일의 최대 개수**를 의미하는 옵션인 것!

보통 Linux 에서 vm.max_map_count 의 기본값은 65530 이다. 하지만 대용량 데이터 처리에 특화된 Elasticsearch 님께서 움직이시는데 고작 65530 개의 메모리 맵 파일로 제대로 된 처리나 할 수 있겠는가. 저 수치 그대로 두고 Elasticsearch 를 초기 구성해서 시작했다간 이런 로그를 받아보기 일쑤이다.

<aside>
📢 [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

</aside>

~~어허 Elasticsearch 님이 움직이신다 메모리 맵 파일을 더 내놓거라~~

## 설정 방법

vm.max_map_count 설정 방법은 어렵지 않다.

### 일시적으로 설정 변경

```bash
sysctl -w vm.max_map_count=262144
```

### 영구적으로 설정 변경

```bash
vi /etc/sysctl.conf
# 아래 1줄을 추가한다
vm.max_map_count=262144
```

### 설정 확인하기

```bash
sysctl vm.max_map_count
```

## 참고

- [https://www.gimsesu.me/elasticsearch-change-vm-max-map-count](https://www.gimsesu.me/elasticsearch-change-vm-max-map-count)
- [https://coder-in-war.tistory.com/entry/Linux-Kernel-메모리-맵-파일mmap](https://coder-in-war.tistory.com/entry/Linux-Kernel-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EB%A7%B5-%ED%8C%8C%EC%9D%BCmmap)
- [http://www.terms.co.kr/paging.htm](http://www.terms.co.kr/paging.htm)
- [https://ko.wikipedia.org/wiki/메모리_맵_파일](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EB%A6%AC_%EB%A7%B5_%ED%8C%8C%EC%9D%BC)
