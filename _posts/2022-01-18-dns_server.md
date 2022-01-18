---
title: "Dear. DNS 서버(a.k.a 반성문)"
excerpt_separator: "<!--more-->"
categories:
  - Infra
tags:
  - DNS
sidebar:
  nav: "docs"
---
예전에 네임 서버 리플레이스 프로젝트를 진행한 적이 있었다. 당시 상사분이 DNS 서버에 대한 매우 길고 유려한(...) 일본어 참고 자료를 보내주셨던 걸 조금 읽다가 도저히 안읽혀서 포기했었다. 이후 DNS 서버의 움직임에 대한 질문에 제대로 답하지 못하는 나에게 해 주신 충고가 아래와 같았다.

“리플레이스 작업이 비록 OS 기초설정과 기존 서버 설정파일 마이그레이션 뿐이라 하더라도, 개념을 이해할 필요가 분명히 있어.”

지당하신 말씀. 너무나도 지당하신 말씀이라 창피할 정도이다. 그러나 받은 자료를 다 읽는 것보다 프로젝트 끝나는 게 더 빠르겠다 여긴 당시의 나는 인터넷에서 DNS 의 흐름을 가볍게 훑고 넘어갔더랬다. ~~그 당시에는 이해한 줄 알았지...~~

그리고 면접 자리에서 대충의 죗값을 톡톡히 치른 후(아... 얼마나 멍청해 보였을까...) 이번에야말로 언제 어디에서든 DNS 서버에 대해 간략하게나마 설명할 수 있을 지식을 머리 속에 넣어놓겠다 다짐하며 작성하는 포스트가 바로 이것. 내가 직접 내용을 써내려가면서, DNS 서버에 대해서는 더 이상 어리버리하게 까먹지 않고 자신있게 말할 수 있게끔. 적어도 잊어버렸을 때 이 포스트를 읽으며 바로 기억을 상기시킬 수 있게끔.

## DNS 서버의 종류

~~과거의 나처럼 a? mx? cname? 이딴 말 지껄이지 말고 기억해두자(참고로 저건 zone 파일에서 사용하는 DNS 레코드의 종류이다)~~

### **Recursive DNS Server**

유저(클라이언트, 이 경우엔 브라우저)가 처음으로 액세스하는 DNS 서버이다.
유저가 도메인을 물어보면(DNS 쿼리를 보내면), 이 서버는 둘 중에 하나의 일을 한다.

(1) 클라이언트가 물어본 도메인에 대한 IP 정보를 모를 경우
보통 이 DNS 서버는 클라이언트와 더 상위의 DNS 서버 사이를 이어주는 중개자 역할을 맡는다.
이 말인즉슨, 서버 본인이 요청온 도메인 네임에 해당하는 IP 주소에 대한 정보를 가지고 있지 않을 경우,
자신보다 상위의 DNS 서버에 해당 도메인 정보를 물어보고, 상위 DNS 서버로부터 IP 정보를 전달받게 되면 그 정보를 다시 클라이언트에 전달해 주는 역할을 한다.

(2) 클라이언트가 물어본 도메인에 대한 IP 정보를 알고 있을 경우
다만, 매번 이 경로를 거치는 것은 매우 비효율적인 일일 것이다(명색의 서버인데 하이패스도 아니고...)
그래서 이 Recursive DNS 서버는 한 번 요청온 DNS 정보에 대해서는 '캐시'를 가져서, 일정 시간(TTL)동안 같은 정보를 다시 요청받으면 상위 DNS 서버를 거치지 않고 자기가 알아서 도메인 정보를 클라이언트에 전달해 준다.

Recursive DNS 서버는 크게 두 종류로 나뉜다.  
(1) ISP DNS 서버 : 통신사 DNS 서버라고 해서, 우리나라의 경우 SK, KT, LG 의 DNS 서버를 의미한다.
(2) Public DNS 서버 : 브라우저 우회 용도로 많이 사용하는 구글 DNS 등을 의미한다.

### **Root DNS Server**

Recursive DNS 서버에 도메인 정보에 대한 캐시가 남아있지 않으면, 클라이언트가 물어본 DNS 쿼리는 이 서버에 도착한다.

여기에는 더 상위의 DNS 서버인 TLD DNS 서버의 IP 정보가 저장되어 있는데, 이 TLD DNS 서버의 정보란, 도메인의 확장자(.com, .net, .org 등)에 따라 다른 TLD DNS 서버 정보를 말한다.

예를 들어 [www.google.com](http://www.google.com/) 이라는 도메인에 대한 쿼리 요청이 들어와서 Root DNS 서버에 도달했다고 하면, Root DNS 서버는 .com 의 정보들만 모여있는 TLD DNS 서버 정보를 Recursive DNS 서버에게 전달해 줄 것이다.

이 Root DNS 서버는 ICANN(DNS 총괄 기구라고...) 이라는 비영리 단체가 관리한다고 하며, 전 세계에 13개의 네임 서버 시스템이 존재한다고 한다(당연하겠지만 진짜 물리 서버가 13대라는 소리는 아님).

### **TLD DNS Server**

Top-Level Domain 의 줄임말로, 최상위 도메인 서버라고도 한다.
위에서 TLD DNS 서버는 도메인 확장자로 구분된다고 쓴 것 처럼, 이 서버는 그 확장자로 끝나는 모든 도메인에 대한 DNS 서버 정보를 가지고 있다.

위에서 예를 든 [www.google.com](http://www.google.com/) 이라는 도메인 정보 요청이 이 TLD DNS 서버까지 도달했다고 하자. 이 요청을 받은 TLD DNS 서버는 분명 .com 확장자를 관리하는 DNS 서버일 것이다. 그럼 이 서버에서는 구글 사에서 관리하는 사설 DNS 서버 정보를 가지고 있을 것이고, 그 사설 DNS 서버를 Recursive DNS 서버에게 전달해 줄 것이다.

TLD DNS 서버는 크게 두 종류로 나뉜다.
(1) 일반 TLD : 말 그대로 일반적인 TLD DNS 서버. .com, .net, .org 등의 확장자에 대한 DNS 서버 정보를 가진다.
(2) 국가별 TLD : 국가별 TLD DNS 서버. .kr, .jp 등의 확장자에 대한 DNS 서버 정보를 가진다.

이 TLD DNS 서버는 위에서 언급된 ICANN 의 지사인 IANA 에서 관리한다고 한다.

### **Authoritative DNS Server**

한국어로 직역하면 권한을 가진 DNS 서버라는 뜻인 Authoritative DNS 서버는, 말 그대로 해당 도메인에 대한 IP 주소를 실제로 가지고 있는 (정보에 대한) 권한을 가진 DNS 서버이다.

TLD DNS 서버는 해당 확장자를 포함하고 있는 모든 도메인에 대해 DNS 서버 정보를 가지고 있다고 했는데, 이 서버가 바로 Authoritative DNS 서버이다. 이 서버는 일반적으로 개인 호스팅 업체에서 관리하는 네임 서버를 의미하며, 크게 잡으면 개인이 운영하는 DNS 서버도 여기에 포함된다.

이 서버에는 해당 호스팅 업체/기업/개인이 사용하는 도메인과 IP 주소의 관계를 적는 'zone' 파일이 존재하는데, Recursive DNS 서버가 이 zone 파일의 내용을 읽어가서 클라이언트에게 요청 받은 도메인 쿼리에 대한 IP 정보를 전달해준다.

실제로 내가 마이그레이션 한 서버가 바로 고객사에서 관리하는 Authoritative DNS 서버인 것이다. 실제로 네임 서버의 정보를 설정하는 곳이 바로 이 Authoritative DNS 서버이기 때문에.

## DNS 서버의 흐름

위의 내용을 쭉 읽어내려오다보면, 클라이언트가 도메인 정보를 어떻게 읽어오는지 이해할 수 있지만, 좀 더 흐름이 잘 보이도록 번호를 붙여서 정리해보도록 한다.

1. 유저가 브라우저에서 www.google.com 을 검색하면, 사용하는 통신사의 Recursive DNS 서버에 IP 주소를 물어보는 도메인 쿼리를 보내게 된다.
2. 여기서 Recursive DNS 서버가 [www.google.com](http://www.google.com) 에 대한 정보를 가지고 있느냐 아니냐에 따라 흐름은 크게 바뀐다.
    1. 만약 Recursive DNS 서버에 www.google.com 에 대한 IP 정보 캐시가 남아있다면, 이 단계에서 바로 브라우저에게 IP 주소를 넘겨주고 업무 종료. 브라우저는 건네받은 IP 주소를 통해 유저를 www.google.com 의 웹페이지로 안내한다.
    2. 만약 Recursive DNS 서버에 www.google.com 에 대한 IP 정보가 없다면, Root DNS 서버에게 물어볼 것이다.
        - Recursive DNS 서버 : (www.google.com 도메인 확장자는 .com 이네) Root DNS 서버야, .com 확장자를 담당하는 TLD DNS 서버 주소 좀 알려줘
        - Root DNS 서버 : 오키오키 (.com TLD DNS 서버 주소를 알려준다)
3. Recursive DNS 서버는 이제 .com 용 TLD DNS 서버에게 물어본다.
    - Recursive DNS 서버 : .com TLD DNS 서버야, www.google.com 정보를 가지고 있는 Authoritative DNS 서버 주소 좀 알려줘
    - TLD DNS 서버 : 오키오키 (구글 사의 Authoritative DNS 서버 주소를 알려준다)
4. Recursive DNS 서버는 드디어 www.google.com 의 IP 주소를 가지고 있는 구글 사의 Authoritative DNS 서버에게 물어본다.
    - Recursive DNS 서버 : 구글 사의 Authoritative DNS 서버야, www.google.com 의 IP 주소 좀 알려줘
    - Authoritative DNS 서버 : 오키오키 (www.google.com 의 IP 주소를 알려준다)
5. Recursive DNS 서버는 열일해서 얻어온 www.google.com 의 IP 주소를 브라우저에게 넘겨주고 업무 종료. 브라우저는 건네받은 IP 주소를 통해 유저를 www.google.com 의 웹페이지로 안내한다.

## 과제

zone 파일의 레코드 개념은 마이그레이션 플젝 하면서 익히기도 했고, 이후에 리눅스마스터 공부하면서 다시 한 번 외운 것도 있어서 설명을 못할 정도는 아니지만(그러고보니 네트워크관리사 시험 공부하면서 다시 한 번 작성해보기도 했네), DNS 관련 지식을 정리하는 김에 추후에 내용을 추가해 두기로 한다.

## 참고

- [https://velog.io/@chun_gil/DNS-서버-정의와-종류](https://velog.io/@chun_gil/DNS-%EC%84%9C%EB%B2%84-%EC%A0%95%EC%9D%98%EC%99%80-%EC%A2%85%EB%A5%98)
- [https://gentlysallim.com/dns란-뭐고-네임서버란-뭔지-개념정리/](https://gentlysallim.com/dns%EB%9E%80-%EB%AD%90%EA%B3%A0-%EB%84%A4%EC%9E%84%EC%84%9C%EB%B2%84%EB%9E%80-%EB%AD%94%EC%A7%80-%EA%B0%9C%EB%85%90%EC%A0%95%EB%A6%AC/)
- [https://www.cloudflare.com/ko-kr/learning/dns/dns-server-types/](https://www.cloudflare.com/ko-kr/learning/dns/dns-server-types/)
- [https://nirsa.tistory.com/108?category=872350](https://nirsa.tistory.com/108?category=872350)
