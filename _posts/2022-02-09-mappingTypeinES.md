---
title: Mapping Type in Elasticsearch
excerpt_separator: "<!--more-->"
categories:
  - Elasticsearch
tags:
  - Mapping Type
  - Index Template
  - Logstash
sidebar:
  nav: "docs"
---

인덱스 템플릿이라고 하면 예전 회사에서 징글징글하게 만들었던 기억이 새록새록... 그렇다고 뭐 대단한 설정을 테스트했던 것까진 아니고, JSON 형식 특유의 중괄호 숫자맞추기로 고생했던 기억이 생생하다.

그 당시 Elasticsearch 는 버전 6.4 정도였어서 mapping 형식도 당연히 version 6 점대에 맞춰서만 알고 있었는데, 이번에 version 7 점대를 대비한 인덱스 템플릿을 준비하는 과정에서 삽질을 꽤 크고 오래 했었고, 오늘 겨우 해결해서 내용을 적어두고자 한다.

사실 개념은 전혀 어려운 것도 아니고, 메이저 버전 6 → 7 로 올라오는 과정에서 변경된 가장 큰 개념인 **mapping type** 과 관련된 부분이라 겸사겸사 공부도 되었다(~~버전 8 데몬이 나온 마당에 이 무슨 말이오 의사양반~~)

우선 시작은 삽질의 이해를 위해 적어보는 이론 이야기부터...

## Mapping Type 이란?

Elasticsearch 버전 6 까지, 인덱스(Index) 를 구성하는 각각의 데이터인 **도큐먼트(Document) 마다 제각각 타입을 정할 수 있었고, 이걸 Mapping Type 이라고** 했다.

인덱스로 묶여있는 도큐먼트에 뭐하러 Mapping Type 을 지정하냐고? 각각의 도큐먼트에 포함된 필드(Field)와 그 필드값(Value) 의 타입은 도큐먼트의 성격(데이터의 종류나 성격)마다 다를 수 있다. 그 도큐먼트의 성격을 타입으로 분류하고, 타입마다 필드와 필드값의 타입을 mapping 할 설정이 지정된다면, **API 등으로 데이터를 검색할 때 인덱스 뿐만 아니라 Mapping Type 까지 지정해서 검색하는 편이 좀 더 효율적이고 빠르게 데이터 결과값을 받아볼 수 있는 방법**이 되지 않겠는가!

그런 이유로 Mapping Type 은 버전 6 점대까지 사용자 지정 형식으로 설정할 수 있었고, 이 값은 **메타 필드인 _type 에 저장**되었다. 또한 이 Mapping Type 은 도큐먼트 사이에서 Parent - Child 관계를 형성할 수도 있었다.

## 왜 Mapping Type 은 사라지게 된 걸까?

저 설명만 보면 Mapping Type 좋은 것 같은데 왜 사라진걸까? 여기에는 좀 복잡한 이유가 있다.

왜 RDBMS 에서는 각각의 테이블이 독립적이지 않나. 예를 들어, 같은 데이터베이스 안에 있더라도, A 라는 테이블의 id 라는 열과, B 라는 테이블의 id 라는 열은 전혀 관련성이 없는 존재인 것처럼.

하지만 ES 는 사정이 다르다. 같은 인덱스 안에 있는 mapping type A 에서 지정한 필드 id 와, mapping type B 에서 지정한 필드 id 는, 내부 구조상 같은 Lucene 필드 안에 백업된다고 한다. 그러니까 이게 다른 mapping type 으로 구분했다고 하더라도 같은 인덱스 안에 있는 같은 이름의 필드면 그 매핑 정보가 동일해야 충돌하지 않는다는 의미이다(~~뭐야 이게... 나누는 이유가 없는데~~).

게다가 mapping type 이 다르더라도 애초에 같은 인덱스 안에 있으면 검색할 때에도 헷갈릴 수 있다고 하니, 대용량 데이터를 다루는 Elasticsearch 에서도 여기까지는 미처 생각하지 못하고 만든 개념이 바로 Mapping type 이 아니었나 생각하게 된다(어쩐지... best practice 에서도 왜 하나의 인덱스에 mapping type 을 여러개 설정하지 말라고 하지 않았나?).

뭐 그래서... 대체안으로 도큐먼트 타입마다 인덱스를 설정하는 방법이라던가, mapping type 과 비슷한 역할을 하되 아예 type 이라는 명시적 필드를 만드는 방법 등이 등장하게 되었다고 한다. 물론, 이전과 같이 Parent - Child 관계를 가진 Mapping Type 은 삭제되었다(대신 Join Field 라는 개념이 생겼다 ~~웬만하면 쓰고 싶지 않다, 데이터 관리와 검색 퍼포먼스에 좋지 않을 것 같아서~~).

뭐... 공식 사이트가 알려준 Mappping Type 개념 얘기는 여기까지 하고, 그래서 삽질한 건 뭐였는데?

## 버전 6 에서 쓰던 Logstash Pipeline 을 바꿔줄 필요가 있다

애초에 내 삽질은, 버전 6에서 쓰던 Logstash Pipeline 을 그대로 버전 7 에 가져와서 적용시켜두고, Elasticsearch 에서 아무리 Index Template 을 만들어도 인덱스가 생성만 되고 데이터가 인덱싱되지 않는 속터지는 상황에서 시작되었다. Index Template 의 Setting 이나 Alias 설정까지는 아무 문제 없이 인입되는 데이터가 Mapping 설정만 추가하면 귀신같이 튕겨져 나가는데 머리에서 스팀이...

게다가 더 답답한 건, Elasticsearch 에서 직접 JSON 으로 샘플 데이터를 PUT 해주면 깔끔하게 인덱싱되는 것. 일단 여기서 문제는 ES 가 아닌 그 전 단계에 있을 것이라 예상해서, 직전 단계인 Logstash 에서 Input 을 다르게 설정해서 테스트해보았다(다른 Input 설정으로 데이터 인입이 성공할 경우 Logstash 전단계인 API 나 Kafka 쪽이 범인, 데이터 인입이 안될 경우 범인은 Logstash 밖에 남지 않을테니). 결과, 다른 Input 으로 집어넣어 본 데이터도 해당 Index Template 앞에서 속절없이 튕겨져 나갔다. 겟챠, 범인은 Logstash Pipeline 이겠구나.

여기서 순간적으로 기시감이 든 게, ES 에서 직접 데이터를 인입시킬 때 썼던 API 였다. 바로 이거.

```json
POST <Index Name>/_doc
```

**_doc** 이라고, 대놓고 7 버전 mapping type 기본값으로 데이터를 인입시켰으니 문제가 될 리 없지. 그렇게 기존 Logstash Pipeline 의 Elasticsearch Output Plugin 문법을 보니까...

```bash
input { ... }
filter { ... }
output {
  elasticsearch {
    hosts               => ["<ES Node #01>", ... "<ES Node #NN>"]
    index               => "<Index Name>"
    document_type       => "<기존 version 6 ES 에서 사용하던 _type 필드값>"  ### 해당 행을 주석처리하거나 삭제하면 version 7 기준 document_type 값이 _doc 으로 통일되어 커스텀 Index template 가 정상적으로 적용된다
  }
}
```

그래, Index Template 의 Mapping 설정이랑 mapping type 을 설정해주는 옵션 document_type 이 애초에 다른데 데이터 인입이 정상적일 리 없지. 게다가,

<div class="notice" markdown="1">
📢 When connected to Elasticsearch 7.x, modern versions of this plugin don’t use the document-type when inserting documents, unless the user explicitly sets `[document_type](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html#plugins-outputs-elasticsearch-document_type)`. If you are using an earlier version of Logstash and wish to connect to Elasticsearch 7.x, first upgrade Logstash to version 6.8 to ensure it picks up changes to the Elasticsearch index template. **If you are using a custom `[template](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html#plugins-outputs-elasticsearch-template)`, ensure your template uses the `_doc` document-type before connecting to Elasticsearch 7.x.**

</div>

... 커스텀한 Index Template 을 쓰려면 얌전히 버전 7의 법칙에 따르라는 말이다.

그리하여 Index Template 만 붙잡고 징글징글하게 테스트(~~삽질~~)하던 걸 버리고 Pipeline 한 줄만 주석처리하자 CLEAR! 데이터가 깨끗하게 인덱싱되기 시작했습니다!

참고로, 이 Mapping Type 은 버전 8 부터는 아예 사라진다고 하니 버전 6 에서부터 쓰던 Logstash Pipeline 을 대대적으로 체크할 필요가 있어보인다.

## 참고

- [https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)
- [https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html#plugins-outputs-elasticsearch)
- [https://yookeun.github.io/elasticsearch/2018/03/09/elastic-mapping/](https://yookeun.github.io/elasticsearch/2018/03/09/elastic-mapping/)
- [https://cloudingdata.tistory.com/109](https://cloudingdata.tistory.com/109)
