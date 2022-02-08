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

{% raw %}<p align="center"><img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-1.png" alt=""></p>{% endraw %}

### 고객이 원하는 인프라 환경은?

1. 인프라가 아닌 애플리케이션의 구축
2. 요구사항에 맞는 인프라 관리 능력
3. 빠르고 원활한 확장(Scale-up 혹은 Scale-out)
4. 보안
→ 자동화를 통한 일관된 보안 유지와 운영 부담 감소!

### 컨테이너와 다른 가상화 환경의 차이는?

베어메탈 서버와 가상머신 환경과 비교한 컨테이너 환경의 차이점은?

{% raw %}<p align="center"><img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-2.png" alt=""></p>{% endraw %}

VM 과 달리 컨테이너는 기본 호스트 시스템의 OS 커널을 공유

## Docker 란?

- 경량 컨테이너 플랫폼
- 이동식 런타임 애플리케이션 환경을 구성할 때 유용
    
    ※ container runtime : 애플리케이션 등이 운영체제 리소스의 분리된 시그먼트를 사용하는 방식
    
- 변하지 않는 단일 아키텍처 안에 애플리케이션을 만드는 데 필요한 모든 종속성 패키지(컨테이너 이미지)를 가진다

### 애플리케이션 환경 구성 요소

{% raw %}<p align="center"><img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-3.png" alt=""><p>{% endraw %}
  

※ Runtime Engine(런타임 엔진) : python, Node.js 등 인터프리터 언어를 실행할 때 필요한 모듈  

Docker 는 런타임 엔진, 종속성, 코드를 **패키지화**하여, 버전 등의 차이에 상관 없이 사용할 수 있도록 해 준다.

{% raw %}<p align="center"><img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-4.png" alt=""></p>{% endraw %}
  

### Image

* 기본 이미지인 **Boot Filesystem** 으로 시작해서 **Root Filesystem** 을 통해 종속성 및 사용자 지정 모드를 추가
* **Dockerfile** 을 통해 쉽게 재현 가능한 빌드가 가능해진다

{% raw %}<p align="center"><img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-6.png" alt=""></p>{% endraw %}
  

* **Boot Filesystem** : 컨테이너를 시작하기 위한 템플릿으로 사용되는 Read-Only 이미지, 상위에 올릴 **Root Filesystem** 이미지와는 상관없이 일정한 설정을 가져간다.
* **Writable Container** : 임시적으로 올려서 사용하는 컨테이너로, 해당 컨테이너가 종료되면 저장되어 있던 데이터는 삭제되므로 별도 백업을 해야 함

※ 레지스트리 : Docker 이미지에 대한 Public/Private 스토리지

### Dockerfile

* Dockerfile 의 각 행은 이미지에 계층을 추가하는 것
* Dockerfile 은 불변하므로, 복수의 Thin R/W Layer 를 몇 번이고 사용 가능하다

아래의 그림과 같이 **CentOS7 install → yum update & httpd(Apache) install → Setting** 으로 이어지는 일련의 초기 구축이 **하나의 Dockerfile 작성을 통해 가능**해진다.

{% raw %}<p align="center"><img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-5.png" alt=""><p>{% endraw %}
  

### Docker 사용의 이점

- 어디서나 안정적으로 실행할 수 있음
- 동시에 다른 애플리케이션을 실행할 수 있음
- 더 나은 자원을 사용할 수 있음

## Microservices Architecture(MSA)

### 기존 아키텍처(모놀리식)과 MSA 의 차이

{% raw %}<img src="https://smilejulie0812.github.io/assets/images/reviewAWSOnlineEdu01-7.png" alt="">{% endraw %}

  
### MSA 의 이점

- 분산 및 진화된 설계가 가능해진다
- 단순한 파이프와 스마트한 Endpoint
- 프로젝트 단위가 아닌 독립된 제품으로 구성 가능
- 장애에 대비한 설계가 가능
- Admin 및 Maintenance 작업을 일회성 프로세스로 실행 가능

## AWS Container Service

### ECS

- **강력한 단순성**
- 규모 있게 컨테이너를 실행할 때 사용 가능한 서비스
- 규모와 기능을 희생하지 않고 의사결정을 감소시킬 수 있음

### EKS

- **열린 유연성**
- AWS 에 최적화된 쿠버네티스 서비스
- 민첩성과 효율성을 확보하며 모든 곳에서 작업을 표준화할 수 있음

### AWS Fargate

- **서버리스 온디멘드 컨테이너**
- 인프라 구성 없이, 컨테이너 수준에서 모든 것을 관리할 수 있는 서비스
- 빠르게 실행할 수 있다는 장점 및 확장의 용이성을 특징으로 지니고 있음
- 리소스에 기반한 요금제 : 사용한 리소스만큼 요금 부과

### AWS Outposts

- 하이브리드 서비스 : 온프레미스 환경과 함께 사용 가능한 컨테이너 서비스

### AWS ECR

- 어디에서나 컨테이너를 관리 및 배포할 수 있는, 100% 클라우드 기반의 Docker 컨테이너 레지스트리

### AWS CodePipeline

- AWS 개발자 도구로 배포 자동화를 가능케 하는 서비스
