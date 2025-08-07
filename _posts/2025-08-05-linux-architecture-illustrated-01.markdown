---
layout: post
title:  "리눅스 개요"
date:   2025-08-05
categories: [linux]
description: 그림으로 배우는 리눅스 구조(타케우치 사토루지음/서수환 옮김) - 한빛미디어
comments: true
bird_image: "frigatebird.webp"
bird_name: "군함조 (Frigatebird)"
bird_scientific_name: "Fregata magnificens"
bird_description: "군함조는 중남미와 갈라파고스 제도 등 열대 및 아열대 해양에서 서식하는 대형 바닷새로, 수컷은 번식기 때 목에 있는 붉은 공기주머니(가슴주머니)를 부풀려 암컷을 유혹한다. 날개 길이가 최대 2.4m에 이르며, 탁월한 비행 능력을 바탕으로 해상에서 장시간 활공할 수 있다. 먹이를 직접 잡기도 하지만, 다른 새를 괴롭혀 먹이를 빼앗는 행동(약탈성 습성)으로도 유명하다."
---

> [그림으로 배우는 리눅스 구조(타케우치 사토루지음/서수환 옮김) - 한빛미디어](https://product.kyobobook.co.kr/detail/S000208795616)

# 리눅스 개요

<img src="{{ '/assets/images/20250807_linux_all.png' | prepend: site.baseurl }}" alt="linux all">

## 프로그램과 프로세스

<img width="60%" src="{{ '/assets/images/20250807_linux_program_process.png' | prepend: site.baseurl }}" alt="linux program process">


프로그램은 정적인 코드와 데이터의 모음이다. 프로세스는 이 프로그램이 실행되어 동작하는 것을 의미한다. 하나의 프로그램은 여러 개의 프로세스를 만들 수 있다.


## 커널

<img width="60%" src="{{ '/assets/images/20250807_linux_kernel.png' | prepend: site.baseurl }}" alt="linux kernel">

커널은 추상화 계층으로 프로세스와 물리적 자원 간의 중간 다리 역할을 수행한다. 

### CPU 모드
x86 아키텍처는 [4개의 모드(Ring 0,1,2,3)](https://en.wikipedia.org/wiki/Protection_ring)를 가지고 있다. 이 중 대부분의 운영체제는 커널 모드(Ring 0)와 사용자 모드(Ring 3)을 활용하여 이원화된 보호 체계를 구축한다. 이러한 권한 분리는 프로세스 간 격리와 시스템 안정성을 보장하고, 프로세스가 시스템 전체에 영향을 미치는 것을 방지한다.

> CPU 아키텍처의 가장 핵심적인 차이는 명령어 집합(ISA)이다.

## 시스템 콜

<img width="60%" src="{{ '/assets/images/20250807_linux_systemcall.png' | prepend: site.baseurl }}" alt="linux systemcall">

사용자 모드에서 커널 모드를 사용하기 위해서는 시스템 콜이 필요하다. 프로세스 생성/종료, 파일 I/O, 네트워크 통신 등 커널 모드의 권한이 필요한 시스템 작업들은 커널의 도움을 받아야 하므로 반드시 시스템 콜을 통해 수행된다.
반면 일반적인 연산(변수 할당, 반복분 실행 등)은 프로세스의 가상 메모리 공간(VIRT) 내에서 처리되어 시스템 콜이 불필요하다.(CPU 사용자 모드로도 접근 가능)  

> 가상 메모리공간에 [페이지 폴트](https://en.wikipedia.org/wiki/Page_fault)가 발생하는 경우에는 하드웨어(MMU)가 감지하여 예외 발생   
> ~~_프로세스가 메모리내놔! 했을 때(시스템 콜) 하드웨어가 없어! 하면서 에러내는 줄 알았는데 아니었다..메모리 내놔가 아니라 이거 읽을게(메모리 접근)~ 하드웨어는 응~ 그거 물리 메모리에 매핑된게 없는데?(페이지폴트) 에러발생~ 이군.._~~

### 라이브러리
사용자 프로그램이 시스템 콜을 호출할 때, 대부분의 OS는 C 언어의 표준 라이브러리인 glibc로 구현된 래퍼 함수를 사용한다.  
이를 통해 어셈블리 코드를 직접 작성하지 않고도, 래퍼 함수를 통해 커널에 시스템 콜을 요청하여 하드웨어 자원에 접근하거나 OS 기능을 사용할 수 있다.

> glibc github - [링크](https://github.com/bminor/glibc)

#### 정적/공유 라이브러리
프로그램이 glibc의 래퍼 함수를 사용하는 방식에는 정적 라이브러리(.a) 방식과 공유 라이브러리(.so) 방식이 있다.
- 정적 라이브러리는  래퍼 함수의 저수준 코드(기계어)가 실행 파일 안에 직접 포함된다.
- 공유 라이브러리는 실행 파일에 래퍼 함수의 위치 정보만 포함하고 실행 시에 동적 로더가 .so 파일을 메모리에 로드한 후,해당 함수 주소를 찾아 동적으로 연결하여 호출한다.

# 실습
## 시스템 콜 확인하기




