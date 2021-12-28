---
title: "Elasticsearch 의 수상한 Swap WARNING 로그"
excerpt_separator: "<!--more-->"
categories:
  - Elasticsearch
tags:
  - Elasticsearch
  - JVM
  - Swap
sidebar:
  nav: "docs"
---
새로 도입한 Data Pipeline 을 통해 Elasticsearch 로그를 받아보면서 평소에 크게 신경쓰지 않았던 내용들을 알게 된다.

아래의 swap 관련 로그 또한 그러한데,

<div class="notice--primary" markdown="1">
📢 cannot compute used swap when total swap is 0 and free swap is 0
</div>

이 로그가 무엇인가 하면,  
기존 JVM 버그(situation when memory.limit_in_bytes == memory.memsw.limit_in_bytes 에서 스왑 공간을 사용하지 않는 경우의 수를 고려하지 않은 탓에  
getFreeSwapSpaceSize()에 의해 음수 값이 반환되는 버그)로 인해 음수 값이 반환되는 경우가 JDK 14 버전을 기준으로 상당히 증가함에 따라 WARNING 레벨로 로그에 출력되는 경우가 매우 잦아졌다는 것이다.

다만 이 로그에 대한 특별한 조치가 필요한가 하면 그건 아니다.  
사용 가능한 메모리나 스왑 공간 중 하나(OR)로 보고되는 통계값이 Elasticsearch 에 의해 0 으로 기록되고 그 영향으로 음수 값이 반환되어 출력되는 로그이지 않은가.  
굳이 사용자가 조치할 필요가 없는 로그이므로 무시하면 그만이다.

Elasticsearch 의 경우 퍼포먼스를 위해 swap 영역을 일부러 잡지 않는 것을 권장받지 않는가.  
이로 인해 **OS 에서 실제로 swap 영역을 잡지 않으면 위의 java 움직임에 의해 getFreeSwapSpaceSize() 가 음수값이 되어 해당 로그가 쉴 새 없이 발생하는 상황**이 벌어지는 것이다.

실제로 관리 중인 Elasticsearch 노드 중 일부 노드(인스턴스:노드=1:1)에서만 해당 로그가 출력되고 있었고, 그 노드는 OS에서 swap 영역이 잡혀있지 않았다.  

```bash
### 혹시 몰라 써두는 swap 영역 확인법
free -h
### 위 명령어로 확인하면 2줄이 출력되는데, 위의 Mem: 이 전체 메모리 사이즈이고
### 아래 Swap: 이 스왑 영역 사이즈이다
```

물론 해당 로그가 매우 신경쓰일 경우에는 다음의 조치를 취할 수 있다.

1. JVM 버그가 수정된 버전으로 업그레이드(아래 정보는 정확하지 않음)
    - JDK 14 이상으로 업그레이드하면 될 것 같은데 테스트해보지는 않았다.
    - JDK 8 의 경우 OpenJDK 8u 로 업그레이드하면 된다고 하는데!
2. Elasticsearch 의 java 소스코드 내에서 해당 로그에 대해 WARNING 으로 출력하는 부분을 DEBUG 로 치환
    
    [https://github.com/elastic/elasticsearch/commit/acd194193a02e3ea741307ea1f05b257db1b75c7](https://github.com/elastic/elasticsearch/commit/acd194193a02e3ea741307ea1f05b257db1b75c7)
    

### 참고

- [https://discuss.elastic.co/t/warn-in-logs-cannot-compute-used-memory-swap/244417](https://discuss.elastic.co/t/warn-in-logs-cannot-compute-used-memory-swap/244417)
- [https://github.com/elastic/elasticsearch/issues/65246](https://github.com/elastic/elasticsearch/issues/65246)
- [https://github.com/elastic/elasticsearch/pull/60375](https://github.com/elastic/elasticsearch/pull/60375)
- [https://bugs.openjdk.java.net/browse/JDK-8242480](https://bugs.openjdk.java.net/browse/JDK-8242480)
