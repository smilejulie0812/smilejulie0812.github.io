---
title: "Network ACL vs Security Group"
excerpt_separator: "<!--more-->"
categories:
  - AWS
tags:
  - VPC
  - NACL
  - Security Group
sidebar:
  nav: "docs"
---
클라우드에 대한 열정은 점점 진심이 되어가고 있는데, 직접 다룰 기회가 많이 주어지지 않는 것은 답답한 일이다. 자격증 공부니 토이플젝이니 야심차게 시작은 했지만 역시 기초 지식부터 많이 부족함을 느끼는 요즘.

VPC 쪽은 그래도 hands-on 에서 간단하게나마 구축해봐서 나름 알고 있다 생각했는데 아주 큰 오산이었다... 이번 포스팅은 NACL, 즉 Network ACL 의 존재조차 모르고 있었던 부끄러움을 극복하기 위해 작성해보도록 한다.

## NACL 이란

NACL 은 Network Access Control List 의 줄임말로, 우리말로 풀어보면 네트워크 액세스 제어 목록이라 한다. 말 그대로 어떤 네트워크 대역에 접속하는 것을 제어하기 위해 설정해 둔 리스트라 할 수 있겠다.
어떤 **네트워크의 서브넷을 기준으로 트래픽(물론 인바운드/아웃바운드 모두)을 제어하기 위한 일종의 방화벽 역할**을 해 주는 것이 NACL 인데, 방화벽의 역할을 한다는 점에서 보통 보안 그룹과 비교되는 개념이다.

## 보안그룹과 NACL 의 차이점을 알고 있나요

개인적으로 AWS 에서 방화벽 역할을 하는 개념은 보안 그룹(Security Group) 밖에 몰랐는데, NACL 과 보안 그룹의 차이점은 무엇일까?

이 내용은 AWS 사용 설명서의 표를 기준으로 정리해본다.

| 차이의 기준 | 보안 그룹 | NACL |
| --- | --- | --- |
| 1. 방화벽을 설정하는 단위(레벨) | **EC2 인스턴스를 기준**으로 방화벽 설정 | **서브넷 범위를 기준**으로 방화벽 설정 |
| 2. 허용/거부 규칙의 지원 | 기본적으로 모든 트래픽이 거부되는 것이 기본값이고, 허용해주는 규칙을 추가함으로써 설정한 규칙에 대해서만 트래픽이 허용되는 구조로, 보안 안정성이 높은 것으로 생각됨 | 허용 규칙과 거부 규칙을 둘 다 추가하는 구조로, 보다 세밀한 규칙을 설정할 수 있음 |
| 3. Stateful vs Stateless | Stateful(트래픽에 대한 상태를 저장) → inbound 규칙을 지정했다면, 그 inbound 로 들어온 트래픽을 다시 출력받을 outbound 규칙을 새로 지정할 필요가 없다는 의미 | Stateless(트래픽에 대한 상태를 저장하지 않음) → inbound 규칙을 지정해서 그 규칙대로 트래픽이 들어왔다고 해도, 해당 트래픽 결과를 출력할 outbound 설정을 별도로 해 주어야 결과를 도출받을 수 있다는 의미 |
| 4. 규칙 적용할 때 순서 | 규칙 적용 순서 없음: 모든 규칙이 동일하게 적용됨 | 규칙 적용 순서 있음: 규칙 설정하는 순서(규칙 번호)를 전략적으로 지정할 필요가 있음 |
| 5. 규칙 적용되는 시기 | 인스턴스를 시작한 후 보안 그룹을 새로 지정하거나, 인스턴스를 시작한 후 기존의 보안 그룹을 연결하거나 등의 별도 적용 절차가 필요 | VPC 안에 NACL 설정이 존재하는 서브넷에 포함되는 기기에 한해서는 별도로 규칙을 설정하거나 규칙을 갱신하지 않아도 자동으로 적용 |

※ 같은 서브넷 범위 내에서 통신할 경우에는 **NACL 보다 Security Group 규칙을 우선**한다.

## 참고

- [https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-network-acls.html](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-network-acls.html)
- [https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_Security.html#VPC_Security_Comparison](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_Security.html#VPC_Security_Comparison)
- [https://cleverdj.tistory.com/122](https://cleverdj.tistory.com/122)
- [https://honglab.tistory.com/153](https://honglab.tistory.com/153)
- [https://medium.com/harrythegreat/aws-가장쉽게-vpc-개념잡기-71eef95a7098](https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098)
