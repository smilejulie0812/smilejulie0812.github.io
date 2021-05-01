---
title: "Java Heap Size와 Elasticsearch의 관계"
excerpt_separator: "<!--more-->"
categories:
  - Elastic
tags:
  - JVM
  - elasticsearch
sidebar:
  nav: "docs"

---
이번 포스팅은 ELK 설정에 빼놓을 수 없는 JVM heap size에 대해 정리해보고자 한다. 시작은 **"JVM Memory에 대해 설명하시오."**라는 한 문장이었지만, 연결되는 내용인만큼 한 번에 정리해 두는 것이 차후 읽기에도 편할 것 같으므로. 
정처기에서는 정의만 가볍게 훑어보고 ELK 구축할 때에는 단순 설정으로 끝난 이 내용을 연관지어 설명함으로써, 머리 속에 다시 한 번 각인해 보도록 한다.

## JVM Memory에 대해 설명하시오
OS 상에는 다양한 프로그램과 어플리케이션이 실행되고, 그 중 많은 프로그램이 Java로 이루어진 Java 어플리케이션일 것이다. 그런데, 이 Java 어플리케이션은 단순히 OS 위에서 돌아가는 것이 아니다. 
OS로부터 Java 어플리케이션을 실행시킬 메모리를 할당받아서 해당 Java 어플리케이션이 실제로 실행되는 공간, 즉 Java 어플리케이션과 OS 사이에서 중계자 역할을 해 주는 곳이 바로 JVM(Java Virtual Machine)이 필요한 것이다. 
그리고, **JVM이 Java 어플리케이션을 실행하기 위해 OS로부터 할당받는 메모리가 JVM Memory**이다.

<div class="notice--info" markdown="1">
**JVM, JRE, JDK 의 의미**  
* **JDK** = JRE + 개발에 필요한 툴  
* **JRE** = JVM + 필요한 라이브러리 = Java 어플리케이션의 실행을 위한 최소 단위  
* **JVM** = OS로부터 Java 어플리케이션을 실행시키기 위한 표준
</div>

## JVM Memory의 구조를 알아야 Java heap size를 이해한다
JVM Memory의 정의를 알았으니, 이번에는 그 구조를 알아보자. 
다만 이 포스팅은 JVM memory의 상세한 구조와 움직임을 파악하려는 것이 아니라 heap size가 무엇인지, 어떤 영향을 주는지 정도를 다룰 것이기 때문에, 이해하기에 필요한 내용만을 픽업해서 적어볼 생각이다.

<구조 그림(추가 예정)>

JVM Memory는 기능에 따라 크게 세 부분으로 나뉘는데, Runtime Data Area, Garbage Collector, Execution Engine이 그것이다.
* **Runtime Data Area**: JVM의 메모리 영역. 즉, OS로부터 메모리 공간을 할당받고 Java 어플리케이션을 실행할 때 실제로 움직이는 공간
* **Garbage Collector(GC)**: Runtime Data Area 내에 존재하는 Heap Area(+Method Area)를 확인해서, 사용하지 않으면서 자리만 차지하는 객체가 존재할 경우, 이를 제거하는 역할
* Execution Engine

그리고 Runtime Data Area는 다시 아래의 다섯 공간으로 나뉜다.
* **Heap Area**: 실행되는 프로그램의 객체와 배열이 저장되는 영역
* Method Area
* Stack Area
* PC Register
* Native Method Stack

자, 여기까지 들어오면 예상이 된다. **Java Heap Size는 바로 Heap Area의 크기**인 것이다. Java Heap Size는 사용자가 지정할 수 있는데, 프로그램이 실행되는 영역이므로 크게 설정하면 좋을 것 같지만 실상 그렇지 않다.

위에서 Heap Area에 저장되어 있는 객체 중 사용되지 않는 객체가 있을 경우 Garbage Collector에 의해 제거된다고 언급했었다. 이 때 Heap Size를 크게 설정한다는 것은 사용되지 않는 객체 또한 저장할 크기가 커진다는 의미이기 때문에, 그만큼 GC가 Heap Area에 관여하는 시간 또한 늘어난다. 즉, Full GC※1의 시간이 증가하고, 그 결과 전체적인 성능이 저하되는 현상이 나타나게 된다. 그러므로 무작정 Heap Size를 늘리기보다는 클러스터링, 로드밸런서 등의 방법을 사용하여 가용성을 확보하는 것이 중요하다.  
한편 Heap Size를 작게 설정한다면 어떻게 될까? 반대로 다루어야 하는 객체를 저장할 공간이 부족해지므로, OOME※2 현상이 발생하게 될 것이다. 그러므로, ~~개복치~~ Java Heap Size을 적절한 크기로 조정하는 것은 매우 중요한 설정이 된다.  
<div class="notice--info" markdown="1">
※1 **Full GC**: Heap Area 전체를 clear하는 작업  
※2 **OOME**: OutofMemory Error의 준말로, Heap Area에서 관리할 수 있는 용량보다 더 많은 Heap size를 요청할 때 생기는 에러
</div>

## 그럼 ES에서 Java Heap Size는 어떻게 설정할까?
이전 포스팅에서도 말했지만, Elastic Stack은 대용량 트래픽을 관리하는 프로그램이므로 메모리 설정이 굉장히 중요하다. 이 Java Heap Size 역시 메모리와 전체 성능에 지대한 영향을 끼치는 설정이므로, Elastic Stack을 인스톨하는 초기 설정에서부터 이를 고려해 주어야 한다.

그렇다면 Elastic Stack 기동을 위해 Java Heap Size를 설정할 때에는 무엇을 고려해야 할까?
* 일반적으로 **Java Heap Size는 RAM의 절반**을 설정한다.
* Elastic Stack에서 **Java Heap Size는 31GB 미만**으로 설정한다.
* **Xms와 Xmx의 크기는 동일**하게 설정한다.

#### Java Heap Size는 RAM의 절반을 설정한다
이유는 간단하다. RAM의 절반 이상을 Java Heap Size로 할당해 버리면, 상대적으로 Lucene이 사용할 메모리가 적어서 그 결과가 성능 저하로 이어지기 때문이다. 그러니 heap size는 RAM의 절반 정도를 설정하는 것으로 생각해 두자.

#### Elastic Stack에서 Java Heap Size는 31GB 미만으로 설정한다
Elastic Stack을 처음 인스톨할 때 Java Heap Size의 디폴트값은 10g이다. 왜 하필 31GB 미만으로 설정해야 하는가는 OOP(Ordinary object pointers)와 깊은 관계가 있는데, 그 부분까지 적게 되면 요점에서 벗어나게 될 거 같으니 패스한다. 그냥 Elastic 관련 노드를 설정할 때의 heap size는 최대 30.5GB를 넘지 않는 것으로 상정한다.

```yml
export ES_HEAP_SIZE=30.5g
```

#### Xms와 Xmx의 크기는 동일하게 설정한다
최소 힙 사이즈(Xms)와 최대 힙 사이즈(Xmx)를 동일하게 해 주는 이유는, 런타임이 일어날 때 heap size가 늘어났다 줄어들었다 하는 과정에서 성능이 저하되는 것을 막기 위해서이다. 웬만하면 같게 맞춰주자.

```yml
ES_JAVA_OPTS="-Xms2g -Xmx2g" ./bin/elasticsearch
```

#### 참고 페이지
* [Heap: Sizing and Swapping](https://www.elastic.co/guide/en/elasticsearch/guide/current/heap-sizing.html#heap-sizing)
* [Advanced configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-heap-size)
* [Setting the heap size](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/heap-size.html#heap-size)
* [자바 메모리 구조 (Java Memory Structure)](https://thinkground.studio/%EC%9E%90%EB%B0%94-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B5%AC%EC%A1%B0-java-memory-structure/)
* [JVM 메모리 구조](https://velog.io/@kyukim/1-yylklo8g)
* [JVM( Java Virtual Machine )이란](https://honbabzone.com/java/java-jvm/#-garbage-collectorgc)
