---
title: "로컬 PC에 CentOS7 환경을 마련하며 생각한 것"
excerpt_separator: "<!--more-->"
categories:
  - Virtualization
tags:
  - VMware Workstation Player
  - Docker Desktop for Windows
  - Oracle VM VirtualBox
  - Windows Subsystem for Linux
sidebar:
  nav: "docs"
---
로컬 PC에 CentOS7 환경을 구축하면서 자잘하게 고민했던 내용을 정리하고자 한다.

리눅스마스터 필기 공부할 때 간단하게 명령어 움직임을 체크하고자 WSL을 설치해서 사용했지만, Redhat 계열 OS 중에서 무료로 풀린 게 없던지라 어쩔 수 없이 Ubuntu를 설치했다. 
이 땐 필기 공부용으로 단순 명령어만 체크하기 위한 용도였으므로 사용하는 데 거슬리는 점은 딱히 없었지만, 슬슬 실기 공부를 해야 할 시기가 왔으므로, 시험 환경을 제대로 만들어두고 공부하고 싶다는 생각을 하게 되었다.

마침 전에 사 둔 네트워크관리사 문제집 부록에 VirtualBox 설치 수순서가 실려있어서 생각 없이 설치해 쓸까 싶었지만, 나름 인프라 엔지니어인데 사용 가능한 여러 툴을 비교해서 골라보고 싶다는 생각에 간단히 검색해 보았다.
다른 방법도 존재하겠지만, 일단 Windows OS 위에서 CentOS를 굴릴 수 있는 가상화 툴은 크게 아래를 찾을 수 있었다.

* VMware Workstation Player를 통해 VM 생성
* Docker Desktop for Windows를 통해 Container 생성
* Oracle VirtualBox를 통해 VM 생성
* Windows Subsystem for Linux를 통해 VM 생성(기존 사용)

일단 WSL를 사용하고 있으니 Hyper-V도 활성화되어 있겠다, 가볍게 Docker라도 사용해서 컨테이너 이미지 생성해서 가볍게 놀려볼까 하다가, 
그래도 vmware랑 나름 친한 사이인데 로컬 환경에 workstation 하나 사용하지 않는 건 너무한가 싶기도 했고, 이거저거 확실히 시험해보기엔 컨테이너보단 머신 쪽이 낫겠지 하는 마음.
무엇보다 스냅숏 기능을 사용할 수 있다는 점이 꽤 매력적이었기에, 
(yum으로 간단히 설치해본다거나 버전업 했을 때 이전 상태로 돌리는 건 굉장히 번거로운 작업이므로 스냅숏으로 이전 상태를 돌릴 수 있다는 건 참 감사한 일이다)
최종적으로는 워크스테이션을 사용하기로 결심(VirtualBox는 음... 스펙만으로 비교했을 땐 역시 좀 달리는 기분이 들어서 크게 고려하지 않았다).  
추가) workstation player는 pro와 달리 무료버전이므로 기본적으로 스냅숏 기능을 제공하지 않으므로 직접 백업 파일을 만들어 사용해야 한다고 한다. 그럼 그렇지(...)

이 즈음에서 가볍게 툴 스펙을 비교하다가 발견한 사실이 있었다. 워크스테이션 15.5 이전 버전까지는 인스톨했을 때 Hyper-V와는 병행해서 사용할 수 없었다는 것이다.
이유인 즉슨, 이전 버전의 워크스테이션은 windows를 **Privileged mode**로 실행해서, CPU에 직접적으로 액세스하여 virtualization 기능을 사용하는 방식이었는데,
Docker를 실행할 경우에는 **Hyper-V mode**를 활성화하여 HW와 OS 사이에 Hypervisor layer를 생성할 필요성이 있으므로, CPU에 직접적으로 접근해야 하는 privileged mode를 사용할 수 없다는 것.

다만, 워크스테이션 버전 15.5부터는 **User mode**에서 Windows Hypervisor Platform API를 사용해서 가상 머신을 실행하는 방식으로 바뀌었다. 워크스테이션 설치 시 WHP를 함께 설치하도록 장려되므로,
WSL이나 Docker와 워크스테이션을 병행하여 사용하고자 하는 사람들에게는 꽤 희소식이 아닐 수 없다(나도 기존에 WSL을 사용하고 있기도 했고, 나중에 Docker 이미지로 fedora를 사용해보고 싶은 욕심도 있었기 때문에 여러모로 도움이 되었다).

그리하여 최종적으로 Workstation에서 설치한 CentOS7. 반갑다 반가워!


{% raw %}<img src="https://smilejulie0812.github.io/assets/images/visualization001-1.PNG" alt="">{% endraw %}

(원래 리눅스의 제맛은 CLI라고 생각해서 X-windows는 별로 내키지 않아하는 1인이나 리눅스마스터 실기 공부를 위해...)

Workstation엔 CentOS7, WSL엔 Ubuntu. 마음이 든든하다. 이제 Docker로 fedora 환경까지 만들면(최신으로 설치해서 CentOS8용 명령어 같은 거 시험해보고 싶다) 가볍게 뛰놀 수 있을만한 리눅스 OS는 대강 갖춰두는 건가.
