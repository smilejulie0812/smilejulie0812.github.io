---
title: Review! AWS Online Edu 「초보자를 위한 컨테이너 시작하기」
excerpt_separator: "<!--more-->"
categories:
  - AWS
tags:
  - AWS Online Education
  - Container
sidebar:
  nav: "docs"
---
회사 메일로 간간이 AWS 관련 정보를 받아보는데, AWS 온라인 교육을 무료로 수강할 수 있는 기회가 있어서 냉큼 신청했다.  
컨테이너 관련 교육이라길래 ECS 랑 EKS 관련 내용이려나 싶었는데 실상은 거의 컨테이너 쌩기초 수업 같은 느낌.  
앞으로 컨테이너 환경에서 인프라를 다룰 일이 무궁무진해질만큼, 한 번 쯤은 짚고 넘어가야 할 내용이라고 생각해서 열심히 수강했다.  
우선 듣고 필기했던 내용으로 정리했지만, 차후에 추가해서 좀 더 의미있는 포스팅으로 완성해 나가도 좋을 듯 하다.

## 컨테이너란?

- 하나의 표준화된 저장 공간 → 효율성을 높이고 간편하며 비용 절감 가능
- Cloud Native Application 을 빠르게 구축하고 관리할 수 있는 도구

{% raw %}<img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-1.png" alt="">{% endraw %}

### 고객이 원하는 인프라 환경은?

1. 인프라가 아닌 애플리케이션의 구축
2. 요구사항에 맞는 인프라 관리 능력
3. 빠르고 원활한 확장(Scale-up 혹은 Scale-out)
4. 보안
→ 자동화를 통한 일관된 보안 유지와 운영 부담 감소!

### 컨테이너와 다른 가상화 환경의 차이는?

베어메탈 서버와 가상머신 환경과 비교한 컨테이너 환경의 차이점은?

{% raw %}<img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-2.png" alt="">{% endraw %}

VM 과 달리 컨테이너는 기본 호스트 시스템의 OS 커널을 공유

## Docker 란?

- 경량 컨테이너 플랫폼
- 이동식 런타임 애플리케이션 환경을 구성할 때 유용
    
    ※ container runtime : 애플리케이션 등이 운영체제 리소스의 분리된 시그먼트를 사용하는 방식
    
- 변하지 않는 단일 아키텍처 안에 애플리케이션을 만드는 데 필요한 모든 종속성 패키지(컨테이너 이미지)를 가진다

### 애플리케이션 환경 구성 요소

<그림>

<그림>

Docker 는 런타임 엔진, 종속성, 코드를 패키지화하여, 버전 등의 차이에 상관 없이 사용할 수 있도록 해 준다.

### Image

* 컨테이너를 시작하기 위한 템플릿으로 사용되는 Read-Only 이미지
* 기본 이미지로 시작해서 종속성 및 사용자 지정 모드를 추가

{% raw %}<img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-5.png" alt="">{% endraw %}

### Dockerfile

Dockerfile 의 각 행은 이미지에 계층을 추가하는 것

### Docker 사용의 이점

- 어디서나 안정적으로 실행할 수 있음
- 동시에 다른 애플리케이션을 실행할 수 있음
- 더 나은 자원을 사용할 수 있음
