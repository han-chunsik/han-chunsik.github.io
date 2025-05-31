---
layout: post
title:  "기술의 흐름으로 이해하는 컨테이너"
date:   2025-05-28
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "monk-parakeet.webp"
bird_name: "퀘이커앵무 (Monk parakeet)"
bird_scientific_name: "Myiopsitta monachus"
bird_description: "퀘이커 앵무새는 작고 활발한 초록빛 앵무새로, 회색 얼굴과 가슴이 특징이다. 남아메리카(특히 아르헨티나) 원산이지만, 미국과 유럽 일부 지역에서도 도입되어 야생 개체군이 형성되었다. 사회성이 강하고 지능이 높아 애완조로 인기가 많으며, 집단 둥지를 짓는 독특한 습성을 가진다."
---

# prologue - 매일 1% 성장하기

**“지금보다 더 나은 엔지니어가 되기 위해 나는 무엇을 해야 할까?”**  

매일 조금씩 성장하기 위해 **인프런 워밍업 클럽 4기 DevOps 과정**에 참여하게 되었다. 해당 과정은 [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1) 강의의 지식공유자가 직접 이끄는 스터디로 매일 강의를 수강하고, 실습하며, 스스로 정리하는 학습 루틴을 통해 지속적인 성장을 돕는 구조이다.
해당 과정을 따라가며 학습한 내용을 꾸준히 정리해나가고자 한다.

## 1% 더 나은 엔지니어가 되기 위해서는?
- 다양한 프로젝트 경험을 통해 근거있는 자신감을 얻는다.
- 내가 이해한 방식으로 큰 그림을 그리고, 새로운 지식을 알게 될 때마다 기존 그림에 추가한다.    
- 주어진 업무 외에도 ‘나에게 필요한 일’을 스스로 정의하고 실행한다.
- 내가 한 일을 잘 정리해두면 지속적인 커뮤니케이션으로 신뢰성을 확보하고, 인수인계에도 도움이 된다.

--- 

# 기술의 흐름으로 이해하는 컨테이너

컨테이너란 무엇일까? 인터넷에 검색해보면 컨테이너에 대한 정의는 다음과 같이 설명되어 있다.
  
> **_"실행중인 컴퓨터의 호스트 운영체제에 격리된 공간을 설정하고, 이 격리공간 내에 호스트 운영체제로부터 독립된 프로세스를 실행시키는 기술과 이를 위한 소프트웨어 구성 일체를 이야기한다. 동일한 하드웨어 아키텍처와 동일한 OS커널을 보유한 수많은 컴퓨터에서 컨테이너로 이미지화한 소프트웨어의 동일한 동작을 보장한다."_**
  
하지만 이 설명을 처음 접했을 때 "그래서 이게 대체 뭔데?" 라는 생각이 먼저 든다. 용어는 복잡하고, 개념은 어렵다. 컨테이너를 왜 써야 하는지, 어떤 문제를 해결해주는지 감이 잘 오지 않는다.      

## 개념의 오해 
다음은 종종 커뮤니티나 실무에서 들을 수 있는 대화이다. 해당 대화에는 몇 가지 오해가 섞여 있다.  
  
```
A: 이제 도커가 유료화된다던데, 이걸 런타임으로 써도 되나요?   
B: 안 되죠. 요즘은 containerd가 대세고, 다 그걸로 넘어가요. 이제 Docker로 되어있는 이미지 다시 만드는 것도 고생 좀 하실 걸요?  
```

1. **"도커가 유료화 된다."**
- Docker Desktop 기업 사용자에 대해 유료 정책이 도입된 것이지 Docker 자체는 오픈소스이며 무료이다.
2. **"Docker는 런타임으로 못쓴다."**
- Docker는 containerd 위에서 동작하는 하나의 클라이언트이다. containerd가 Kubernetes의 기본 CRI(Runtime Interface)로 채택되었을 뿐 런타임으로 계속 사용 가능하다.
3. **"Docker로 만든 이미지를 다시 만들어야 한다."**
- Docker 이미지 포맷은 OCI(Open Container Initiative) 표준을 따른다. Docker로 만든 이미지는 containerd, CRI-O 등 다른 런타임에서도 그대로 사용할 수 있다.

### 왜 이런 오해가 생길까?
기술의 겉모습만 알거나 지금 대세인 기술만 쫓아가면 기술이 등장한 배경이나 구조를 모르기 때문에 오해하기 쉽다.   
따라서 기술을 흐름으로 공부하면 오해를 줄일 수 있다. 기술이 어떤 문제에서 출발했고, 어떤 과정을 거쳐 지금의 모습이 되었는지를 이해하면, 단순한 오해를 넘어서 진짜 선택을 할 수 있게 된다.

## 리눅스 배포판 흐름
<img src="{{ '/assets/images/20250528_linux_flow.png' | prepend: site.baseurl }}" alt="linux flow">

컨테이너는 리눅스 커널 기능을 기반으로 동작하기 때문에, 사용하는 리눅스 배포판에 따라 구성 방식이나 운영 특성에 차이가 생긴다. 이를 제대로 이해하기 위해선 각 배포판이 어떻게 등장하고 발전해왔는지를 함께 살펴보는 것이 중요하다.

- 초기 운영체제인 **UNIX**는 유료였고, 이를 대체하기 위해 **Linux**가 오픈소스로 등장했다.

### 대표적인 계열 구분
- **Debian 계열**: Ubuntu  
- **RedHat 계열**: RHEL, CentOS, Fedora, AlmaLinux, RockyLinux


#### RedHat 계열의 흐름
- **Fedora**: 최신 기술 실험용, 빠른 릴리즈 주기
- **RHEL**: 기업용 상용 배포판, 안정성 중시
- **CentOS**: RHEL 복제판으로 인기를 끌었으나, 2021년부터 테스트 성격의 CentOS Stream으로 전환됨
- **Rocky/Alma**: CentOS 대안으로 등장한 커뮤니티 기반 RHEL 복제판

> IBM이 RedHat을 인수한 이후, CentOS의 방향이 바뀌었고, 그 역할을 현재는 **RockyLinux**나 **AlmaLinux**가 이어가고 있다.

> #### Kubernetes를 설치할 때도 이 계열 구분이 중요한 기준이 된다.    
> - Ubuntu 기반 설치, RHEL 계열 설치에서 **패키지 매니저, SELinux 정책, 서비스 구성 방식**이 다르기 때문이다.  
> - 오픈소스 기술 선택 시에는 GitHub 스타 수, Google Trends 등의 **수치 기반 지표**를 참고하는 것도 좋은 방법이다.

## Container 흐름
<img src="{{ '/assets/images/20250529_container_flow.png' | prepend: site.baseurl }}" alt="container flow"> 

> CRI-O는 Red Hat이 주도하여 개발한, Kubernetes 전용 컨테이너 런타임이다.

컨테이너는 리눅스 커널이 제공하는 여러 격리 기능들을 조합한 것으로, 대표적으로 다음 세 가지 핵심 기술이 있다.

- **chroot**: 파일 시스템을 특정 디렉토리 기준으로 제한하여, 외부 파일에 접근하지 못하게 하는 기술  
- **cgroup**: CPU, 메모리 등 시스템 자원의 사용량을 제한하고 분리하는 기능  
- **namespace**: 프로세스 ID, 네트워크, 사용자 등 다양한 커널 리소스를 독립된 공간으로 분리하는 기능

이 기능들을 조합하면 격리된 실행환경을 만들 수 있지만, 너무 복잡하다는 단점이 있었다.

### Container 도구
#### LXC(Linux Containers)
리눅스의 격리 기능들을 하나로 묶어 컨테이너 환경을 구성할 수 있도록 만든 것이 LXC이다.  
최초의 컨테이너 런타임으로, 리눅스 커널 기능을 직접 다루지 않고도 컨테이너를 만들 수 있었다. 하지만 여전히 설정이 복잡하고 사용자 친화적이지 않아 시스템 관리자나 리눅스 전문가가 아니면 쓰기 어려운 도구였다.  

#### Docker
Docker는 LXC 위에 편리한 인터페이스와 도구들을 얹어 컨테이너를 누구나 쉽게 빌드하고 실행할 수 있도록 만든 플랫폼이다.   
초창기에는 Docker 내부에서 LXC를 런타임으로 사용했다. 하지만 더 세밀한 제어와 독립성을 위해 Docker는 자체 런타임인 libcontainer를 만들었고, 이후에는 이를 개선한 containerd와 runc로 구조를 분리하였다.

> #### LXC, Docker 차이
> - LXC: 리눅스 OS 수준의 컨테이너 관리, 운영체제를 컨테이너화하는 것이 목적
> - Docker: 애플리케이션 실행, 앱을 쉽고 빠르게 배포하는 것이 목적  
>  
> LXC는 OS를 위해, Docker는 App을 위해 만들어졌다.

### 왜 LXC보다 Docker가 더 널리 쓰이게 되었을까?

LXC는 커널 수준에서 컨테이너를 다루는 저수준 도구로, 컨테이너 생성부터 네트워크 설정, 파일 시스템 마운트까지 대부분의 작업을 사용자가 직접 수동으로 설정해야 했다. 반면 Docker는 이러한 기능들 위에 고수준 인터페이스를 제공하며, dockerd와 같은 데몬 및 명령어 기반의 CLI를 통해 누구나 쉽게 컨테이너를 생성하고 배포할 수 있도록 지원한다.  

<img width="60%" src="{{ '/assets/images/20250529_docker_engine.png' | prepend: site.baseurl }}" alt="docker engine">  

이러한 사용 편의성과 자동화된 부가 기능 덕분에 Docker는 LXC보다 훨씬 더 널리 사용되는 컨테이너 플랫폼으로 자리잡게 되었다.
  
> **저수준(low level)**: 기계 친화적, 추상화 정도가 낮아 세부 제어가 가능하지만 복잡도가 높고, 제어에 어려움이 있다.(예: C, Assembly, LXC 등)  
> **고수준(high level)**: 사람 친화적, 추상화 정도가 높아 복잡도가 낮고, 제어가 비교적 쉽다.(예: Python, Java, Docker 등)  


## 컨테이너와 컨테이너 오케스트레이션의 관계
### 컨테이너 오케스트레이션의 필요성
서비스가 커지고 그에따라 컨테이너 수가 많아지면서 컨테이너의 생명주기를 자동으로 관리해줄 시스템(운영 자동화)이 필요해졌다. 이러한 요구 속에서 등장한 것이 컨테이너 오케스트레이션이다. 대표적인 오케스트레이터가 바로 Kubernetes 이다.

### 서비스 업그레이드 과정 비교
#### 컨테이너만 사용하는 경우
모든 작업이 수동이며, 사람이 직접 컨테이너의 상태와 네트워크를 관리해야 한다.  

<img width="60%" src="{{ '/assets/images/20250529_container_upgrade.png' | prepend: site.baseurl }}" alt="container upgrade">  

#### 컨테이너 오케스트레이션을 사용하는 경우
생명주기, 상태 관리, 트래픽 전환까지 모두 자동화 된다.  

<img width="60%" src="{{ '/assets/images/20250529_container_orchestration_upgrade.png' | prepend: site.baseurl }}" alt="container orchestration upgrade">  

### "Kubernetes에서 Docker가 빠진다"는 무슨 말인가?
Kubernetes는 1.24부터 더 이상 Docker를 컨테이너 런타임으로 사용하지 않는다.   

기존의 Kubernetes는 아래와 같은 과정을 거쳐 컨테이너를 실행했다. 이 구조에서 dockershim은 Kubernetes와 Docker사이의 중간 어댑터 역할을 했지만 불필요한 오버헤드가 존재했다.  

```      
Kubernetes → dockershim → Docker → containerd → 컨테이너 실행
```  

Kubernetes 1.24부터 dockershim이 제거되고 아래와 같이 container 런타임을 직접 호출하는 구조로 변경되었다.

```
Kubernetes → containerd → 컨테이너 실행
```
   
결국 "Kubernetes에서 Docker가 빠진다" 라는 말은 Kubernetes클러스터에서는 containerd가 직접 컨테이너를 관리하므로 Docker 데몬이 불필요해졌다는 뜻이다.

> Docker 자체도 내부적으로 containerd를 사용한다.

# Kubernetes 버전에 따른 컨테이너 생성 흐름 변화
Kubernetes는 버전별로 컨테이너 런타임과 통신하는 방식이 바뀌어왔다. 

<img src="{{ '/assets/images/20250531_kubernetes_create_container_flow.png' | prepend: site.baseurl }}" alt="container orchestration upgrade">  

# 회고
이번 학습을 통해 가장 크게 느낀 점은, 기술을 단편적인 기능이 아니라 흐름을 보며 이해해야 진짜 의미가 보인다는 것이다.  
컨테이너 기술이 왜 등장했고, 어떤 한계를 해결했으며, 지금은 어떤 구조로 진화해왔는지 그 흐름을 따라가다 보면 자연스럽게 각각의 기술이 존재하는 이유가 보이는거 같다.
  
또 하나 인상 깊었던 점은, 컨테이너의 개념은 크게 달라지지 않았지만 내부 구조가 꾸준히 바뀌어 왔다는 것이다. 추상화가 있기에 사용자는 복잡한 내부를 몰라도 생산성을 높일 수 있고, 내부는 더 유연하고 효율적인 방향으로 변화할 수 있다.
  
앞으로도 기술을 배울 때는, 흐름 속에서 이해하려는 습관을 가져야겠다.