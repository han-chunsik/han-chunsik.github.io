---
layout: post
title:  "배포 파이프라인 구축 - jenkins"
date: 2025-06-22
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "albatross.webp"
bird_name: "알바트로스 (Albatross)"
bird_scientific_name: "Diomedeidae"
bird_description: "알바트로스는 세계에서 가장 큰 날개폭(최대 3.5m)을 가진 바닷새로, 주로 남반구의 바람이 강한 바다 위를 활공하며 살아간다. 날갯짓보다 활공에 의존해 수천 km를 쉬지 않고 비행할 수 있고, 바다 위에서 오랜 시간을 보내며 오징어나 물고기를 먹고 산다. 평생 한 짝과만 짝짓기를 하며, 긴 수명과 느린 번식 속도가 특징이다. 뛰어난 비행 능력과 우아한 모습 때문에 바다의 전설적인 새로도 불린다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)

> 출처: [큐브옵스 - Jenkins Pipeline (기초부터 Blue/Green까지)](https://cafe.naver.com/f-e/cafes/30725715/articles/87?boardtype=L&menuid=13&referrerAllArticles=false&page=2)  
> 본 글은 출처를 참고하여 테스트를 진행한 결과를 바탕으로 작성하였습니다. 설치 및 실습 과정은 원문을 참고하시고, 이 글에서는 핵심 개념 정리와 추가 설명에 집중하였습니다.


# 배포 파이프라인 구축 실습 - jenkins
배포 파이프라인을 구축하는 데에는 다양한 도구들이 활용된다. 이번 실습에서는 여러 도구 중 Jenkins를 활용해 배포 파이프라인을 구성해보고 Blue/Green 배포 방식을 실습해볼 예정이다.  
실무에서는 보통 ArgoCD와 같은 툴로 블루그린 배포를 자동화하지만, 이번에는 스크립트를 직접 작성하여 수동 배포 과정을 이해하는 데 집중한다.

## 실습 순서
1. Jenkins Pipeline 기본 구성 만들기
2. GitHub와 연동 및 파이프라인 단계 세분화
3. Blue/Green 배포 구현 및 특징 실습
4. 스크립트를 통한 자동 배포 구성


## 1. Jenkins Pipeline 기본 구성 만들기
Jenkins에서 agent는 파이프라인의 작업을 어디서 실행할지 지정하는 실행환경이다. 각 stage 또는 전체 파이프라인 단위로 설정할 수 있으며, 대표적인 유형은 다음과 같다.

- agent any : 사용 가능한 모든 노드에서 실행 (Master, Slave 무관)
- agent label('노드이름') : 지정한 레이블의 노드에서만 실행
- agent docker { image '이미지' } : 지정한 Docker 이미지로 컨테이너에서 실행
- agent dockerfile : 현재 프로젝트의 Dockerfile로 이미지 빌드 후 실행

## 2. Github 연결 및 파이프라인 세분화

<img src="{{ '/assets/images/20250622_jenkins_pipeline_test.png' | prepend: site.baseurl }}" alt="jenkins pipeline test">

해당 실습에서는 전체 배포 과정이 여러 단계(Stage) 로 나뉘어 구성되어 있음을 확인할 수 있다. 이처럼 배포 과정을 세분화하면 다음과 같은 장점이 있다.  

- 문제 발생 지점 파악: 각 단계별로 로그가 구분되므로, 어느 단계에서 오류가 발생했는지 명확하게 확인할 수 있다.
- 시간이 오래 걸리는 구간 식별: 빌드, 테스트, 배포 등 단계별 소요 시간을 확인함으로써 병목 지점을 찾는 데 도움이 된다.
- 특정 단계 선택적 재시도: 전체 파이프라인을 다시 실행하지 않고, 실패한 단계만 다시 실행할 수 있어 효율적이다.
- 파이프라인 구조의 가독성 향상: 각 단계가 명확히 나뉘어 있어 유지보수가 쉬워지고, 협업 시에도 이해하기 쉬운 구조를 제공한다.

## 3. Blue/Green 배포 만들기
블루그린 배포는 일반적으로 자원을 2배 사용한다고 알려져 있다. 실제로 앱이 동시에 두 환경(blue와 green)에서 기동되기 때문에 CPU 사용량은 일시적으로 증가하는 것이 맞다. 그러나 트래픽이 하나의 버전으로만 흐르게 되므로 지속적으로 CPU가 2배 사용되지는 않는다.
반면, 메모리는 두 환경 모두에서 지속적으로 점유되므로 실제로 2배 사용된다고 보는 것이 타당하다.

### 배포 설계 시 고려할 요소
- Deployment 네이밍 전략: 블루그린 배포를 원활하게 진행하려면 배포 리소스의 이름에 버전이나 시퀀스를 붙이는 방식으로 관리할 필요가 있다. 예를 들어 my-app-v1, my-app-v2와 같은 네이밍을 활용한다.

- Label 및 Selector 관리: 트래픽이 두 환경에 동시에 전달되지 않도록 하기 위해, 서비스와 디플로이먼트에 적절한 label 및 selector 설정이 필요하다. 이를 통해 명확하게 트래픽 전환 대상을 지정할 수 있다.


### 수동 배포 절차
1. Green 배포 생성: 새로운 Green 버전의 Deployment를 생성하고, 이를 테스트하기 위한 별도의 Service도 함께 만든다.
2. 서비스 트래픽 전환: 기존 Blue 서비스를 중단하고, Service의 selector를 Green 쪽으로 변경하여 트래픽을 전환한다.
3. 정리 및 롤백 대비: Blue Deployment 및 불필요한 리소스를 삭제하거나, 문제가 발생할 경우 빠르게 롤백할 수 있도록 레이블 정보를 관리한다.

> 수동으로 레이블을 일일이 변경하면 누락될 가능성이 높기 때문에, Helm과 같은 도구를 활용하면 변경 누락 없이 안정적인 배포가 가능하다.

> kubectl create는 리소스가 이미 존재하면 실패하기 때문에, 기존 리소스를 업데이트하거나 새로 생성할 수 있는 kubectl apply를 사용하는 것이 일반적이다. 또한, 특정 속성만 부분적으로 수정할 때는 kubectl patch를 활용한다.

### 자동 배포 절차
1. Green Deployment 생성: 새로운 버전의 Deployment(Green)를 생성한다.
2. Pod 상태 확인: Green 환경의 Pod가 Ready 상태가 되었는지 확인한다.
3. Service 트래픽 전환: Service의 selector를 Green으로 변경하여 트래픽을 전환한다.
4. Blue Deployment 제거: 이전 버전(Blue)의 Deployment를 삭제하고, 관련된 레이블이나 리소스 정보를 정리한다.

# 회고
이번 주 강의에서는 데브옵스를 구성하는 핵심 요소들과 전체적인 흐름을 배우고, 그중 배포 관련 실습을 진행했다. 이 과정을 통해 개발자가 오직 개발에만 집중할 수 있는 환경을 만드는 것이 데브옵스의 중요한 역할임을 체감했고, 그 과정 자체가 더 흥미롭게 느껴졌다.  
또한 단순히 이론만 공부하는 것이 아니라, 계속 직접 테스트 환경을 구축하고 실습을 진행하다 보니 테스트에 대한 자신감도 생겼다. 환경 구성에 익숙해지면서, 앞으로 새로운 기술이나 툴을 접하더라도 빠르게 테스트해볼 수 있겠다는 확신이 들었다.


